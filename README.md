# Setup Hashicorp Vault Server on Docker

Sample project to setup a Vault Server on Docker and demonstrate how to get started with the Vault CLI to initialize the vault, create, use and remove secrets.

## General syntax for vault commands

In general a vault command follows this rule:
```
vault module operation arguments
```

Here are some of the most commonly used modules with some of their operations

* operator : init, unseal, rekey
* audit : enable
* auth : enable
* secrets : enable
* kv : put, get, list, enable-versioning, delete, metadata
* policy : read, write

There is also a global anonymous module with operations like login, status, read, write...

## Commands sequence

The following command have to be executed on the container's terminal

1. `export VAULT_ADDR='http://127.0.0.1:8200'`
2. `vault operator init -key-shares=6 -key-threshold=3`
3. `vault operator unseal <UNSEAL KEY>` (3 times)
4. `vault status -format=json`
5. `vault login <TOKEN>`
6. `vault audit enable file file_path=/vault/logs/vault_audit.log`
7. `vault secrets enable -version=2 -path=secret kv`
8. `vault kv put secret/my-app/password password=123`
9. `vault kv list secret/`
10. `vault kv get secret/my-app/password`
11. `vault kv get --format=json secret/my-app/password`
12. `vault kv get -field=password secret/my-app/password`
13. `vault kv put secret/reminders/app db_username=db.ruanbekker.com username=root password=secret`
14. `vault kv get --format=json secret/reminders/app`
15. `vault kv get -field=username secret/reminders/app`
16. `vault kv delete secret/reminders`
17. `vault kv metadata put -max-versions=5 secret/fooapp/appname`
18. `vault kv metadata get secret/fooapp/appname`
19. `vault kv put secret/fooapp/appname appname=app1`
20. `vault kv put secret/fooapp/appname appname=app2`
21. `vault kv put secret/fooapp/appname appname=app3`
22. `vault kv get -field=appname secret/fooapp/appname`
23. `vault kv get -field=appname -version=2 secret/fooapp/appname`

## Remarks

* The invocation of the `vault operator init` command will display 6 unseal keys and an initial root token. The number of unseal keys displayed depends on the **key-shares** parameters.

* The `vault operator unseal` has to be invoked 3 times because 3 is the value supplied to the **key-threshold** parameter of the `vault operator init` command. The expected unseal key is one of those produced by `vault operator init`.

* While invoking the `vault login command` a token is required. That token is supplied by vault after the call of the `vault operator init` command after the list of unseal keys. Look for a line starting with _Initial Root Token_.

  **e.g.:** 
  ```sh
  Initial Root Token: hvs.DZcQGslNrf2fWMXBnNP7UYcW
  ```

## Original article link

The entire process is well explained on this page: 
[Setup Hashicorp Vault Server on Docker and a Getting Started CLI Guide](https://blog.ruanbekker.com/blog/2019/05/06/setup-hashicorp-vault-server-on-docker-and-cli-guide/)

## Jenkins integration

### Enable AppRole and the kv-v2 engine

```sh
vault auth enable approle
vault secrets enable kv-v2
vault kv enable-versioning secret/
```

### Create a policy for your approle: KV Secrets Engine — Version 2

```sh
vault policy write jenkins /vault/config/jenkins-policy.hcl
vault write auth/approle/role/jenkins token_policies="jenkins" token_ttl=1h token_max_ttl=4h
```

### Get RoleID and SecretID

```sh
vault read auth/approle/role/jenkins/role-id
vault write -f auth/approle/role/jenkins/secret-id
```

### Create github secret with 3 keys to read in jenkins pipeline

```sh
vault kv put secret/jenkins/github @/vault/config/github.json
```

### Read vault’s secrets from Jenkins declarative pipeline

The rest of the process is to be handled on Jenkins as described on this [page](https://codeburst.io/read-vaults-secrets-from-jenkin-s-declarative-pipeline-50a690659d6).
