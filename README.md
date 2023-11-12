# Setup Hashicorp Vault Server on Docker

Sample project to setup a Vault Server on Docker and demonstrate how to get started with the Vault CLI to initialize the vault, create, use and remove secrets.

## Commands sequence

The following command have to be executed on the container's terminal

1. `export VAULT_ADDR='http://127.0.0.1:8200'`
2. `vault operator init -key-shares=6 -key-threshold=3`
3. `vault operator unseal <UNSEAL KEY>` (3 times)
4. `vault status -format=json`
5. `vault login <TOKEN>`
6. `vault secrets enable -version=1 -path=secret kv`
7. `vault kv put secret/my-app/password password=123`
8. `vault kv list secret/`
9. `vault kv get secret/my-app/password`
10. `vault kv get --format=json secret/my-app/password`
11. `vault kv get -field=password secret/my-app/password`
12. `vault kv put secret/reminders/app db_username=db.ruanbekker.com username=root password=secret`
13. `vault kv get --format=json secret/reminders/app`
14. `vault kv get -field=username secret/reminders/app`
15. `vault kv delete secret/reminders`
16. `vault kv metadata get secret/fooapp/appname`
17. `vault kv put secret/fooapp/appname appname=app1`
18. `vault kv put secret/fooapp/appname appname=app2`
19. `vault kv put secret/fooapp/appname appname=app3`
20. `vault kv get -field=appname secret/fooapp/appname`
21. `vault kv get -field=appname -version=2 secret/fooapp/appname`

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
tee jenkins-policy.hcl << EOF
path "secret/data/jenkins/*" {
    capabilities = [ "read" ]
}
EOF

vault policy write jenkins jenkins-policy.hcl

vault write auth/approle/role/jenkins token_policies="jenkins" token_ttl=1h token_max_ttl=4h
```

### Get RoleID and SecretID

```sh
vault read auth/approle/role/jenkins/role-id
vault write -f auth/approle/role/jenkins/secret-id
```

### Create github secret with 3 keys to read in jenkins pipeline

```sh
tee github.json << EOF
{
	"private-token": "76358746321876543",
	"public-token": "jhflkweb8y7432",
	"api-key": "80493286nfbds43"
}
EOF

vault kv put secret/jenkins/github @github.json
```

### Read vault’s secrets from Jenkins declarative pipeline

The rest of the process is to be handled on Jenkins as described on this [page](https://codeburst.io/read-vaults-secrets-from-jenkin-s-declarative-pipeline-50a690659d6).
