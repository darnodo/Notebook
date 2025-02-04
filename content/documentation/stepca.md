---
title: "Self-Hosted Certificat Manager"
date: 2024-08-01T20:00:00+02:00
cascade:
  type: docs
---

## Sources

- [Official Documentation](https://smallstep.com/docs/tutorials/)
- [Step-Ca as a systemd service](https://angrysysadmins.tech/index.php/2022/09/grassyloki/step-ca-run-as-a-systemd-service/)
- [OpenSSL Certificate Management](https://www.golinuxcloud.com/tutorial-pki-certificates-authority-ocsp/)

## About Step-CA

Step-CA is a set of tools developed by Smallstep, a company specializing in secure identity management and certificate automation. Step-CA aims to simplify the process of setting up and managing certificate authorities (CAs) by providing a user-friendly and secure infrastructure.

The Step-CA toolkit offers several key features:

1. Certificate Authority Management: Step-CA allows users to easily set up and manage their own certificate authorities. It provides tools for creating root and intermediate CAs, issuing certificates, and managing certificate revocations.

2. Secure Key Management: Step-CA incorporates best practices for secure key management. It offers secure storage and management of cryptographic keys, ensuring their protection against unauthorized access and tampering.

3. Automation and Scalability: Step-CA supports automation and scalability, making it suitable for both small-scale and large-scale deployments. It provides APIs and integrations that allow for automated certificate issuance, renewal, and revocation, streamlining the certificate lifecycle management process.

4. Enhanced Security: Step-CA prioritizes security and employs modern cryptographic algorithms and protocols. It supports industry-standard X.509 certificates and provides strong encryption and digital signatures to ensure the integrity and authenticity of certificates.

5. Integration with Infrastructure: Step-CA integrates seamlessly with existing infrastructure and tools. It supports various authentication mechanisms, including traditional username/password, multi-factor authentication, and integration with external identity providers.

6. Auditability and Compliance: Step-CA offers comprehensive logging and auditing capabilities, allowing organizations to track and monitor certificate issuance and management activities. This helps meet compliance requirements and enables investigation in case of security incidents.

7. Developer-Friendly APIs: Step-CA provides developer-friendly APIs and SDKs, enabling easy integration into custom applications and workflows. This allows developers to incorporate certificate management capabilities into their software with minimal effort.

In summary, Step-CA from Smallstep is a toolkit designed to simplify the management of certificate authorities. It offers secure and scalable infrastructure for setting up and managing CAs, with a focus on automation, security, and integration. With its user-friendly features and developer-friendly APIs, Step-CA enables organizations to efficiently manage their certificate lifecycle and enhance the security of their infrastructure.

## Installation

### Binairy Installation

1. Step CLI :

    ```bash
    wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
    sudo dpkg -i step-cli_0.24.3_amd64.deb
    ```

2. Step-ca :

    ```bash
    wget https://dl.step.sm/gh-release/certificates/docs-ca-install/v0.24.1/step-ca_0.24.1_amd64.deb
    sudo dpkg -i step-ca_0.24.1_amd64.deb
    ```

3. Create a specific user

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

Generating root certificate... done!
Generating intermediate certificate... done!

✔ Root certificate: /home/adminCA/.step/certs/root_ca.crt
✔ Root private key: /home/adminCA/.step/secrets/root_ca_key
✔ Root fingerprint: 7d754397c6897aa87d21e33c64daad7be087dc6fe18bf04627848ae1c8e26a4f
✔ Intermediate certificate: /home/adminCA/.step/certs/intermediate_ca.crt
✔ Intermediate private key: /home/adminCA/.step/secrets/intermediate_ca_key
✔ Database folder: /home/adminCA/.step/db
✔ Default configuration: /home/adminCA/.step/config/defaults.json
✔ Certificate Authority configuration: /home/adminCA/.step/config/ca.json

Your PKI is ready to go. To generate certificates for individual services see 'step help ca'.

FEEDBACK  
  The step utility is not instrumented for usage statistics. It does not phone
  home. But your feedback is extremely valuable. Any information you can provide
  regarding how you’re using `step` helps. Please send us a sentence or two,
  good or bad at feedback@smallstep.com or join GitHub Discussions
  https://github.com/smallstep/certificates/discussions and our Discord 
  https://u.step.sm/discord.
```

Start CA Step :

```bash
step-ca .step/config/ca.json
```

#### Enable ACME

```bash
$ step ca provisioner add acme --type ACME
✔ CA Configuration: /home/adminCA/.step/config/ca.json

Success! Your `step-ca` config has been updated. To pick up the new configuration SIGHUP (kill -1 <pid>) or restart the step-ca
 process.$
```

#### Run Step-CA as a systemd service

Create a file :

```bash
vim /etc/systemd/system/step-ca.service
```

Copy and paste :

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

Create log dir :

```bash
mkdir -p /var/log/step-ca
chown -R adminCA:adminCA /var/log/step-ca
```

Reload Daemon :

```bash
systemctl daemon-reload
systemctl start step-ca.service
```

### Docker Installation

```bash
docker run -it -v step:/home/step \
    -p 9000:9000 \
    -e "DOCKER_STEPCA_INIT_NAME=Lab" \
    -e "DOCKER_STEPCA_INIT_DNS_NAMES=caserver.lab.loc,localhost,192.168.1.101 \
    -e "DOCKER_STEPCA_INIT_REMOTE_MANAGEMENT=true" \
    -e "DOCKER_STEPCA_INIT_ACME=true" \
    smallstep/step-ca
```

## Access To CA with an another client

> [!NOTE] Note
> It is necessary to adapt the port according to the installation:
>
> - Binary: port **443**
> - Docker: port **9000**

```bash
wget https://dl.step.sm/gh-release/cli/docs-cli-install/v0.24.3/step-cli_0.24.3_amd64.deb
sudo dpkg -i step-cli_0.24.3_amd64.deb

step ca bootstrap --ca-url https://caserver.lab.loc:$PORT/ --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
```

>[!TIP] Tips  
> In order to obtain the **fingerprint** it is enough to use the following command:
>
> ```bash
> step certificate fingerprint $(step path)/certs/root_ca.crt
> ```
>
> For Docker : check the container logs

```bash
admin@User:~$ step ca bootstrap --ca-url https://caserver.lab.loc:$PORT --fingerprint 685059c30eb305db5272a7a199a2b5823624d55c732121ac65c06b0915d3c887
The root certificate has been saved in /home/admin/.step/certs/root_ca.crt.
The authority configuration has been saved in /home/admin/.step/config/defaults.json.
```

And install :

```bash
step certificate install $(step path)/certs/root_ca.crt
```

---

>[!TIP] Tips  
> **Debian Installation:**
>
> - Copy individual CRT (in PEM format) files to `/usr/local/share/ca-certificates/`
> - Files must be own bu `root:root` and mode `644`
> - Check that the package `ca-certificates` is installed, if not install it
> - Then run as root :
>
> ```bash
> # /usr/sbin/update-ca-certificates
> ```
>
> - All the certificates will be consolidated at : `/etc/ssl/certs/ca-certificates.crt`
>

---

### Get a certificat

```bash
admin@User:~$ step ca certificate nas.lab.loc srv.crt srv.key
✔ Provisioner: contact@lab.loc (JWK) [kid: chyGkrZqp-BGSHUZ8v3jsPipegt2JLcC7y6RPq4OOkU]
Please enter the password to decrypt the provisioner key: 
✔ CA: https://caserver.lab.loc:443
✔ Certificate: srv.crt
✔ Private Key: srv.key
```

---
>[!TIP] Tips  
>To health check :
>
>```bash
>curl https://caserver.lab.loc:443/health -k
>```
>

---

It can be necessairy to customise `ca.json` file to  increase the minimum duration of the certificate validity.

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
