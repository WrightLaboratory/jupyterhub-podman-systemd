# Instructions for Running `main.yml` Playbook

## Setting Up Control Node to Create jupyterhub

### Create `nginx-reverse-proxy-ssl-vars.yml`

Generate a self-signed certificate and key using openssl and gzip them into an archive named `{{ fqdn_of_service }}.gz` or follow the instructions [here](https://github.com/vbalbarin/cert-manager/tree/dev-vbalbarin) to create a certificate and key bundled with a root CA and intermediate key.

(The latter method will allow you or your users to go past the SSL warnings in Chromium based browsers.)

```
export TARGET_FQDN={{ fully_qualified_domainname_of_server }}
export TARGET_USER={{ name_of_account_on_target_with_sudo }}

tar -xvf ${TARGET_FQDN}.gz -C ./roles/nginx-reverse-proxy/files
```

### Generate Jupyterhub Secrets

```bash
ansible-playbook -i localhost, --connection local jupyterhub-secrets-generate.yml
```

Answer the prompts

Use `vi` to edit  `${HOME}/.ansible/vaultpassword` and replace the line with your plaintext password.
(Be sure not to add a new line.)

Encrypt resulting the `${HOME}/.ansible/jupyterhub-secrets.yml` file

```bash
ansible-vault encrypt --vault-id $(id -un)@${HOME}/.ansible/vaultpassword ${HOME}/.ansible/jupyterhub-secrets.yml
```

### Create Jupyterhub

```bash
ansible-playbook --user="${TARGET_USER}" \
    --vault-id "$(id -un)@${HOME}/.ansible/vaultpassword" \
    -e "@${HOME}/.ansible/jupyterhub-secrets.yml" \
    -i "${TARGET_FQDN}", \
    "main.yml"
 ```
