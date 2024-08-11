# Gover Helm Chart
## Prerequisites
- A Kubernetes cluster
- A Server running PostgreSQL to store your data
    - You can install an example on a server you have access via SSH to, with the included Ansible playbook in the `./services` directory
- A Server running MinIO to store your files and uploaded attachments
    - You can install an example on a server you have access via SSH to, with the included Ansible playbook in the `./services` directory
- A Server running ClamAV where the ClamAV daemon socket is exposed to check uploaded files for viruses
    - You can install an example on a server you have access via SSH to, with the included Ansible playbook in the `./services` directory
- Helm3 installed to deploy this Helm Chart
- Nginx Ingress Controller installed in your cluster (Only if you want to use the nginx ingress setup with automatic certificate generation)
- Cert Manager installed in your cluster (Only if you want to use the nginx ingress setup with automatic certificate generation)

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
Create a values file `./my-values.yml` based on the content of [./values.yml](./values.yml).

Make sure to setup the values for MinIO, ClamAV and PostgreSQL correctly.

## Install the Gover Helm Chart
Install the gover helm chart with the following command:

```bash
helm install --namespace=YOUR_NAMESPACE --create-namespace --values=./my-values.yml INSTALL_NAME ./gover
```

## Setup Keycloak and Gover
You can use the realm exports from [https://github.com/aivot-digital/gover/tree/main/realm-templates](https://github.com/aivot-digital/gover/tree/main/realm-templates) to set up the Keycloak realms.
Make sure to follow the instructions in the [README.md](https://github.com/aivot-digital/gover?tab=readme-ov-file#setup-keycloak) of the repository to set up the realms correctly.

When the realms are imported set the GoverBackendClientSecret in the values file.
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

## Uninstall the Gover Helm Chart
Uninstall the Gover helm chart with the following command:

```bash
helm uninstall \
  --namespace=<YOUR_NAMESPACE> \
  <INSTALL_NAME>
```