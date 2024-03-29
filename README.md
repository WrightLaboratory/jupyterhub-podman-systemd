# Containerized JupyterHub under Rootless Podman Systemd

## Introduction

[JupyterHub](https://jupyter.org/hub) provides for a centrally managed, multi-user JupyterLab environment.

The recommended deployment requires access to a Kubernetes environment for largescale production workloads.
One may also use Dockercompose.
The Littlest JupyterHub provides access for small laboratory environments.

This solution aims to provide a JupyterHub environment at small to medium scales on a single node.

Ansible and Podman, unlike Dockercompose, allows for migration from a single node to a Kubernetes environment with minor modifications.

Note, this solution uses Github Oauth for authentication.
Configure a new GitHub account under your organization using these [instructions](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app).
You will need this information to setup the `jupyterhub-secrets.yml` file.

## Prepare Ansible Management Node

Clone this repository:

```
git clone https://github.com/WrightLaboratory/jupyterhub-podman-systemd.git jupyterhub-podman-systemd

cd jupyterhub-podman-systemd
```

Install Python using [`pyenv`](https://github.com/pyenv/pyenv).

```
pyenv install $(cat .python-version)

python -m venv ./.venv

source ./.venv/bin/activate

pip install -r requirements.txt
```

Pull the required Ansible roles from the the Wright Laboratory `ansible-roles` repository as a Git submodule:

```
git submodule add  https://github.com/WrightLaboratory/ansible-roles.git roles

# Install required Ansible modules
ansible-galaxy install -r ./roles/requirements.yml
```

## Running Playbook

Generate Ansible vault file containing JupyterHub configuration and secrets:

```
ansible-playbook -i localhost, --connection local jupyterhub-secrets-generate.yml

ansible-vault encrypt --vault-id $(id -un)@${HOME}/.ansible/vaultpassword ${HOME}/.ansible/jupyterhub-secrets.yml
```

Prepare environment for running playbook.

```
export TARGET_FQDN="{{ FQDNOfJupyterHubServer }}"
export TARGET_USER="{{ NameOfUserWithAdminPrivilegesOnServer }}"
```

Generate an archive that contains an SSL certficate, key, and bundle file using this [repository](https://github.com/vbalbarin/cert-manager.git).
(Note, you can generate and use a self-signed certificate and key without a certificate authority chain but Chromium-based browsers will not allow you to bypass the certificate error.)

Extract this archive to `./roles/nginx-revers-proxy/files`

```
mkdir -p ./roles/nginx-reverse-proxy/files
tar -xvf ${TARGET_FQDN}.gz -C ./roles/nginx-reverse-proxy/files
```

Execute playbook:

```
ansible-playbook --user="${TARGET_USER}" \
    --vault-id "$(id -un)@${HOME}/.ansible/vaultpassword" \
    -e "@${HOME}/.ansible/jupyterhub-secrets.yml" \
    -i "${TARGET_FQDN}", \
    main.yml
```