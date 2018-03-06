# Security
## Some Home Office Application Container Platform resources related to Security

Some resources which may be useful:

### KeyCloak:
[https://github.com/UKHomeOffice/application-container-platform/blob/master/docs/keycloak.md](https://github.com/UKHomeOffice/application-container-platform/blob/master/docs/keycloak.md)

*   Worth confirming with Home Office Architecture team if this is a plain vanilla KeyClock image.
*   Note that the instructions on the above page refer to using MySQL RDS so this will also need discussing with HO Architecture (i.e. changing it to PostGres RDS).
*   I don’t know whether KeyCloaks existing user login (it also includes registration) and user admin can be re-used. Worth asking HO Architecture how other services have made use of KeyCloak

### Application Composition:
[https://github.com/UKHomeOffice/application-container-platform/blob/master/docs/application.md](https://github.com/UKHomeOffice/application-container-platform/blob/master/docs/application.md)

*   Mentions supporting proxy containers such as KeyClock and Nginx.
*   The Vault container is mentioned. This is used for handling secrets and certificated. It is marked  as deprecated so it would be useful to understand if there is an alternative for secrets management (this service sounds like it might be quite useful). Note that the proxy fronts an Open Source

### Here are some ‘How To Docs’: [https://github.com/UKHomeOffice/application-container-platform/blob/master/how-to-docs/README.md](https://github.com/UKHomeOffice/application-container-platform/blob/master/how-to-docs/README.md)

*   In some of the linked documents pages there is some mention of Vault for secrets management
