# Linux Certificate Autoenrollment in Active Directory Domain
Linux clients can be configured to automatically enroll user and machine certificates (e.g. for VPN connection) from a Windows CA server, similar as it can be configured for Windows clients.

This can be done by using [cepces](https://github.com/openSUSE/cepces) and [certmonger](https://pagure.io/certmonger). The CEP/CES protocol/feature role needs to be installed on the Windows CA server.

For user certificates, you need at least cepces version 0.4.x and a valid kerberos ticket for the user. A kerberos ticket is automatically aquired when logging in with a domain account on the Linux client. See [Linux Domain Join](<Linux Domain Join.md>) for more information.

- [Machine Certificate](https://github.com/openSUSE/cepces#user-certificates)
- [User Certificate](https://github.com/openSUSE/cepces#user-certificates)
