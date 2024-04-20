# Gover Helm Chart
## Prerequisites
- A Kubernetes cluster
- Helm3 installed
- Nginx Ingress Controller installed (Only if you want to use the nginx ingress setup with automatic certificate generation)
- Cert Manager installed (Only if you want to use the nginx ingress setup with automatic certificate generation)

Clone this chart into the directory `./gover`:

```bash
git clone https://github.com/aivot-digital/gover-chart.git ./gover
```

### Install Cert Manager
You can install the Cert Manager with the following command:

```bash
helm install \
  cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --create-namespace \
  --version v1.14.4 \
  --set installCRDs=true
````

## Create a values file
Create a values file `./my-values.yml` with the following content and overwrite all necessary values:

```yaml
# The hostname where Gover will be available
Host: my.gover.digital

# Determine if the namespace of this chart should be used as a node selector
# This is useful if you run several instances of Gover on the same cluster and want to separate the setups onto physically different nodes
# If you set this to true, the namespace will be used as a node selector
NamespacedNodes: true

# Set this to true to disable the ingress and the cert manager setup for Gover
# This is useful if you want to use your own ingress setup or if your cluster supports any other method of ingress / load balancing
DisableIngress: false

# Settings for the Gover instance
Gover:
  # Version of the Gover image
  Version: 4.0.1

  # URL where the Gover instance will be available
  Hostname: https://my.gover.digital

  # Mail address where errors reports will be sent to
  ReportMail: sysadmin@aivot.de

  # Mail address that will be used as the sender for mails
  FromMail: '"My Gover System" <noreply@my.gover.digital>'

  Sentry:
    # The environment that will be sent to Sentry for error tracking
    Environment: development

    # The DSN for the Sentry instance where the backend errors will be sent to
    Server: https://eada765f8e614b54b6f2d251fd69e252@sentry.aivot.cloud/2

    # The DSN for the Sentry instance where the frontend errors will be sent to
    App: https://1677139e6e374703aaed37a824e3fb4a@sentry.aivot.cloud/3

  Files:
    # List of allowed file extensions
    Extensions: |
      pdf, png, jpg, jpeg, doc, docx, xls, xlsx, ppt, pptx, odt, fodt, ods, fods, odp, fodp, odg, fodg, odf

    # List of allowed file content types
    ContentTypes: |
      application/pdf, image/png, image/jpeg, application/msword, application/vnd.openxmlformats-officedocument.wordprocessingml.document, application/msexcel, application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/mspowerpoint, application/vnd.openxmlformats-officedocument.presentationml.presentation, application/vnd.oasis.opendocument.text, application/vnd.oasis.opendocument.spreadsheet, application/vnd.oasis.opendocument.presentation, application/vnd.oasis.opendocument.graphics, application/vnd.oasis.opendocument.formula

# Settings for the Keycloak instance
Keycloak:
  # Version of the Keycloak image
  Version: 24.0.1.1

  # Hostname where Keycloak will be available
  Hostname: my.gover.digital

  # Initial username of the Keycloak admin user
  Username: admin

  # Initial password of the Keycloak admin user
  Password: admin

  # The secret of the keycloak backend client which will be used by Gover to authenticate against Keycloak
  # Set this value after setting up the staff realm in Keycloak
  GoverBackendClientSecret: secret

  # Settings for the identity provider plugins
  # This data is needed for communication with the BundID, BayernID, Serviceportal Schleswig-Holstein and Mein Unternehmenskonto
  IdProviderData:
    # Name of the Gover instance when interacting with an identity provider
    Name: Gover

    # Name of the Gover instance displayed in the identity provider
    DisplayName: Gover Testinstanz

    # Description of the Gover instance displayed in the identity provider
    Description: Testinstanz von Gover

    # Organization data which will be sent to the identity provider
    Org:
      Name: Musterstadt
      Url: https://musterstadt.de

    # Technical contact data which will be sent to the identity provider
    TechnicalContact:
      Name: Max Mustermann
      Email: technik@musterstadt.de

    # Support contact data which will be sent to the identity provider
    SupportContact:
      Name: Erika Musterfrau
      Email: hilfe@musterstadt.de


# Settings for the Gover database
GoverDatabase:
  Version: 15.6-alpine3.18
  Username: gover
  Password: gover

# Settings for the Keycloak database
KeycloakDatabase:
  Version: 15.6-alpine3.18
  Username: keycloak
  Password: keycloak

# Settings for the Redis cache
Redis:
  Version: 7.2.4
  Database: 0

# Settings for the ClamAV AntiVirus
ClamAV:
  Version: 1.3.0

# Settings for the SMTP server
SMTP:
  Host: smtp.relay.de
  Port: 587
  Username: sysadmin@aivot.de
  Password: passwort
```

## Install the Gover Helm Chart
Install the gover helm chart with the following command:

```bash
helm install \
  --namespace=<YOUR_NAMESPACE> \
  --create-namespace \
  --values=./my-values.yml \
  <INSTALL_NAME> \
  ./gover
```

## Setup Keycloak
You can use the realm exports `./files/export-customer-realm.json` and `./files/export-staff-realm.json` to setup the Keycloak realms.
Just log into the Keycloak admin console and import the realms.

## Set Backend Client Secret
After setting up the staff realm in Keycloak, you need to set the backend client secret for Gover in the values file.
This is because the backend client secret needs to be set for the backend to correctly interact with the Keycloak.
Just regenerate the secret in the Keycloak admin console for the client called `backend` in the staff realm and set it in your values file.
Update the value for `Keycloak.GoverBackendClientSecret` in your values file and run the following command:

```bash
helm upgrade \
  --namespace=<YOUR_NAMESPACE> \
  --values=./my-values.yml \
  <INSTALL_NAME> \
  ./gover
```