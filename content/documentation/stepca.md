---
title: "Self-Hosted Certificate Manager"
date: 2024-08-01T20:00:00+02:00
cascade:
  type: docs
---

## 🔗 Sources

- [📖 Official Documentation](https://smallstep.com/docs/tutorials/)
- [🛠️ Step-CA as a systemd Service](https://angrysysadmins.tech/index.php/2022/09/grassyloki/step-ca-run-as-a-systemd-service/)
- [🔐 OpenSSL Certificate Management](https://www.golinuxcloud.com/tutorial-pki-certificates-authority-ocsp/)

## 🤖 About Step-CA

Step-CA is a nifty toolkit developed by Smallstep, a company that’s all about secure identity management and certificate automation. 🚀 Its mission? To simplify setting up and managing your own certificate authorities (CAs) with ease and security!

### Key Features

1. **Certificate Authority Management** 🔑  
   Easily set up and manage your own CAs. Create root and intermediate CAs, issue certificates, and handle revocations like a pro.

2. **Secure Key Management** 🛡️  
   Best practices for secure key storage and management, ensuring your cryptographic keys stay safe and sound from unauthorized access.

3. **Automation and Scalability** ⚙️  
   Perfect for both small-scale and enterprise deployments. Enjoy APIs and integrations that automate certificate issuance, renewal, and revocation for a seamless lifecycle.

4. **Enhanced Security** 🔒  
   Using modern cryptographic algorithms and protocols, Step-CA supports industry-standard X.509 certificates, offering robust encryption and digital signatures.

5. **Integration with Infrastructure** 🌐  
   Integrates smoothly with your existing tools and systems. Supports various authentication methods like username/password, MFA, and external identity providers.

6. **Auditability and Compliance** 📜  
   With comprehensive logging and auditing capabilities, you can track certificate activities and meet compliance requirements with ease.

7. **Developer-Friendly APIs** 👩‍💻👨‍💻  
   Developer-centric APIs and SDKs make it a breeze to integrate certificate management into your custom applications and workflows.

**In a nutshell:** Step-CA from Smallstep is designed to make certificate authority management fun and hassle-free. With its secure, scalable, and user-friendly features, you can easily manage your certificate lifecycle while keeping your infrastructure safe and sound!

## 🚀 Installation

### 🔧 Binary Installation

#### 1. Step CLI

```bash
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
sudo dpkg -i step-cli_0.24.3_amd64.deb
```

#### 2. Step-CA

```bash
wget https://dl.step.sm/gh-release/certificates/docs-ca-install/v0.24.1/step-ca_0.24.1_amd64.deb
sudo dpkg -i step-ca_0.24.1_amd64.deb
```

#### 3. Create a Specific User

```bash
adduser adminCA
```

#### Configuration

```bash
$ step ca init --password-file=password.txt
✔ Deployment Type: Standalone
What would you like to name your new PKI?
✔ (e.g. Smallstep): Lab
✔ (e.g. ca.example.com[,10.1.2.3,etc.]): ca.lab.loc, localhost, 192.168.1.101
What IP and port will your new CA bind to? (:443 will bind to 0.0.0.0:443).1.101
✔ (e.g. :443 or 127.0.0.1:443): :443
What would you like to name the CA's first provisioner?
✔ (e.g. you@smallstep.com): contact@lab.loc
Choose a password for your CA keys and first provisioner.
✔ [leave empty and we'll generate one]: 

Generating root certificate... done! 🎉
Generating intermediate certificate... done! 🎊

✔ Root certificate: /home/adminCA/.step/certs/root_ca.crt  
✔ Root private key: /home/adminCA/.step/secrets/root_ca_key  
✔ Root fingerprint: 7d754397c6897aa87d21e33c64daad7be087dc6fe18bf04627848ae1c8e26a4f  
✔ Intermediate certificate: /home/adminCA/.step/certs/intermediate_ca.crt  
✔ Intermediate private key: /home/adminCA/.step/secrets/intermediate_ca_key  
✔ Database folder: /home/adminCA/.step/db  
✔ Default configuration: /home/adminCA/.step/config/defaults.json  
✔ Certificate Authority configuration: /home/adminCA/.step/config/ca.json  

Your PKI is all set! To generate certificates for individual services, check out `step help ca`.

💌 **FEEDBACK**  
The step utility is not instrumented for usage statistics. It doesn’t phone home. But your feedback is super valuable! Feel free to drop us a line at feedback@smallstep.com, join GitHub Discussions, or hop into our Discord at [https://u.step.sm/discord](https://u.step.sm/discord).
```

Start CA Step:

```bash
step-ca .step/config/ca.json
```

#### Enable ACME

```bash
$ step ca provisioner add acme --type ACME
✔ CA Configuration: /home/adminCA/.step/config/ca.json

Success! Your `step-ca` config has been updated. To pick up the new configuration, SIGHUP (kill -1 <pid>) or restart the step-ca process. 🎉
```

#### Run Step-CA as a systemd Service

Create a file:

```bash
vim /etc/systemd/system/step-ca.service
```

Copy and paste the following:

```config
[Unit]
Description=step-ca
After=syslog.target network.target

[Service]
User=adminCA
Group=adminCA
ExecStart=/bin/sh -c '/bin/step-ca /home/adminCA/.step/config/ca.json --password-file=/home/step/.step/pwd >> /var/log/step-ca/output.log 2>&1'
Type=simple
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Create the log directory:

```bash
mkdir -p /var/log/step-ca
chown -R adminCA:adminCA /var/log/step-ca
```

Reload the daemon:

```bash
systemctl daemon-reload
systemctl start step-ca.service
```

### 🐳 Docker Installation

```bash
docker run -it -v step:/home/step \
    -p 9000:9000 \
    -e "DOCKER_STEPCA_INIT_NAME=Lab" \
    -e "DOCKER_STEPCA_INIT_DNS_NAMES=caserver.lab.loc,localhost,192.168.1.101" \
    -e "DOCKER_STEPCA_INIT_REMOTE_MANAGEMENT=true" \
    -e "DOCKER_STEPCA_INIT_ACME=true" \
    smallstep/step-ca
```

## 🔑 Access to CA with Another Client

> **NOTE:**  
> Adjust the port based on your installation:  
>
> - **Binary:** port **443**  
> - **Docker:** port **9000**

Install the Step CLI:

```bash
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
sudo dpkg -i step-cli_0.24.3_amd64.deb
```

Bootstrap your CA:

```bash
step ca bootstrap --ca-url https://caserver.lab.loc:$PORT/ --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
```

> **TIP:**  
> To get the **fingerprint**, simply run:
>
> ```bash
> step certificate fingerprint $(step path)/certs/root_ca.crt
> ```
>
> For Docker, check the container logs.

Example output:

```bash
admin@User:~$ step ca bootstrap --ca-url https://caserver.lab.loc:$PORT --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
The root certificate has been saved in /home/admin/.step/certs/root_ca.crt.
The authority configuration has been saved in /home/admin/.step/config/defaults.json.
```

Install the certificate:

```bash
step certificate install $(step path)/certs/root_ca.crt
```

---

> **TIP:**  
> **Debian Installation:**  
>
> - Copy individual CRT (PEM format) files to `/usr/local/share/ca-certificates/`  
> - Files must be owned by `root:root` with mode `644`  
> - Ensure the package `ca-certificates` is installed (if not, install it)  
> - Then run as root:
>
> ```bash
> # /usr/sbin/update-ca-certificates
> ```
>
> All certificates will be consolidated at: `/etc/ssl/certs/ca-certificates.crt`

---

### 📝 Get a Certificate

```bash
admin@User:~$ step ca certificate nas.lab.loc srv.crt srv.key
✔ Provisioner: contact@lab.loc (JWK) [kid: chyGkrZqp-BGSHUZ8v3jsPipegt2JLcC7y6RPq4OOkU]
Please enter the password to decrypt the provisioner key:
✔ CA: https://caserver.lab.loc:443
✔ Certificate: srv.crt
✔ Private Key: srv.key
```

---

> **TIP:**  
> To perform a health check:
>
> ```bash
> curl https://caserver.lab.loc:443/health -k
> ```

---

It might be necessary to customize the `ca.json` file to increase the minimum duration of the certificate validity. Check out the folder structure below:

```bash
.
|-- certs
|   |-- intermediate_ca.crt
|   `-- root_ca.crt
|-- config
|   |-- ca.json
|   `-- defaults.json
|-- db
|   |-- 000000.vlog
|   |-- 000020.sst
|   |-- KEYREGISTRY
|   |-- LOCK
|   `-- MANIFEST
|-- secrets
|   |-- intermediate_ca_key
|   |-- password
|   `-- root_ca_key
`-- templates
```

Example `ca.json` file:

```json
{
    "root": "/home/step/certs/root_ca.crt",
    "federatedRoots": null,
    "crt": "/home/step/certs/intermediate_ca.crt",
    "key": "/home/step/secrets/intermediate_ca_key",
    "address": ":9000",
    "insecureAddress": "",
    "dnsNames": [
        "caserver.lab.loc",
        "caserver",
        "localhost",
        "192.168.1.200"
    ],
    "logger": {
        "format": "text"
    },
    "db": {
        "type": "badgerv2",
        "dataSource": "/home/step/db",
        "badgerFileLoadingMode": ""
    },
    "authority": {
        "enableAdmin": true,
        "claims": {
            "maxTLSCertDuration": "4380h"
        }
    },
    "tls": {
        "cipherSuites": [
            "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256",
            "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256"
        ],
        "minVersion": 1.2,
        "maxVersion": 1.3,
        "renegotiation": false
    }
}
```
