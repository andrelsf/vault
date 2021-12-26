# Vault

Learning about Vault Hashcoorp

## Init Environment

```bash
docker-compose up -d
docker-compose exec vault /bin/sh
```
---
## Initial Commands

```bash
vault version
vault status

# Initializing and Unsealing
vault operator init

# unseal Vault using three of the keys
vault operator unseal

# Login with root key
vault login
```
---
### Enable Auditing

- [Vault Audit](https://www.vaultproject.io/docs/audit)

```shell script
vault audit enable -description=Auditlogs file file_path=/vault/logs/audit.logs
vault audit list
```
---
### Key-Value Engine

```shell script
vault secrets enable -path=multicode -description="KV Secrets for project multicode" -version=2 kv
vault kv enable-versioning multicode

vault secrets list

# create or update a value for a key
vault kv put multicode/secrets key=VAIII
vault kv get multicode/secrets
vault kv metadata get multicode/secrets
vault kv put multicode/secrets key=FUIII
vault kv metadata get multicode/secrets
vault kv get -version=1 multicode/secrets
vault kv get -version=2 multicode/secrets
vault kv delete -versions=1 multicode/secrets
vault kv metadata get multicode/secrets
vault kv undelete -versions=1 multicode/secrets
vault kv get -version=1 multicode/secrets
vault kv destroy -versions=1 multicode/secrets
vault kv put multicode/secrets email=test.email@gmail.com password=123456789
vault kv get -field email multicode/secrets
vault kv get -field password multicode/secrets

# Forma de destruir versions
vault kv destroy -versions=1,2,3 multicode/secrets
```
---

## Auth Methods

* [Auth Methods](https://www.vaultproject.io/docs/auth)

```bash
vault auth <options>

# Por padrão o tipo token já esta habilitado
vault auth list

# Habilitando e desabilitando o tipo userpass
vault auth enable userpass
vault auth disable userpass

# Criando para um PATH especifico
vault auth enable -path=multicode userpass
vault write auth/multicode/users/andrelsf password=multicode
vault read auth/multicode/users/andrelsf

# Autenticando o usuario
vault login -method=userpass -path=multicode username=andrelsf password=multicode

# Autenticando via TOKEN
vault login token=<USER_TOKEN>

# Removendo usuário
vault delete auth/multicode/users/andrelsf
vault auth disable multicode
```

`Nota:` Neste processo o Vault irá gerar um token que pode ser usado para autenticar no via TOKEN.

---
## Policies

* [Vault Policies](https://learn.hashicorp.com/tutorials/vault/policies)

`PATHs`: Dentro do contexto do Vault PATHs são pontos de montagem quando um Secret é habilitado.

Policies são maneiras de aplicar ACLs (Access Control List) há determinados PATHs, assim ter controle de quem acessa e o que pode ser operado dentro do contexto das Secrets Engine.

As policies escritas no formato HCL são chamadas de Policies ACL.

Na inicialização do Vault é criada uma policy default. Por padrão tudo e bloqueado (Denied).

Além de definir as permissões, ainda deve ser realizada a definição do tipo de ação que podem ser operadas em determinados PATHs.

Capabilities:
- create
- read
- update
- delete
- list
- patch
- sudo
- deny


```json
//"multicode/data/secrets" json only
// multicode-ro-policy.hcl
path "multicode/secrets" {
    capabilities = [ "read" ]
}
```

```bash
vault secrets enable -path=multicode -description="KV Secret engine for project Multicode" kv
vault kv put multicode/secrets username=andre.ferreira password=123456789

vault policy list
vim multicode-ro-policy.hcl
vault policy write multicode-ro-pl ./multicode-ro-pl.hcl

vault auth enable -path=multicode -description="Auth userpass for Multicode Project" userpass
vault write auth/multicode/users/andrelsf password=multicode policies=multicode-ro-pl
vault read auth/multicode/users/andrelsf

vault login -method=userpass -path=multicode username=andrelsf password=multicode

vault kv get multicode/secrets
vault kv put multicode/secrets key=UPDATED_VALUE

vault login token=<ROOT_TOKEN>

vault policy list
vault policy delete multicode-ro-pl

vault auth list
vault auth disable multicode
vault delete auth/multicode/users/andrelsf

vault secrets disable multicode
```
---
## Transit Secrets Engine

* [Data Encryption: EAAS](https://learn.hashicorp.com/tutorials/vault/eaas-transit?in=vault/encryption-as-a-service)
* [App Integration with Spring Vault](https://learn.hashicorp.com/tutorials/vault/eaas-spring-demo?in=vault/app-integration)

```bash
# Configure
vault secrets enable -path=crypto-srv transit

# Create an encryption key ring
vault write -f crypto-srv/keys/mysecretkey

# Policy definition 
vim vault/policies/crypto-srv-transit-update.hcl
```

`Policy`
```json
path "crypto-srv/encrypt/mysecretkey" {
  capabilities = [ "update" ]
}

path "crypto-srv/decrypt/mysecretkey" {
  capabilities = [ "update" ]
}
```

```bash
# Create a policy named multicode-crypto-pl.
vault policy write multicode-crypto-pl /vault/policies/crypto-srv-transit-update.hcl

# Create a token with multicode-crypto-pl attached.
vault token create -policy=multicode-crypto-pl
```

`Client httpie: POST Encrypt`
```bash
export VAULT_TOKEN="<TOKEN_CLIENT>"

# Encode Base64
echo "1111222233334444" | base64

vim payloads/to-encrypt-data.json
```

`payload`
```json
{
  "plaintext": "MTExMTIyMjIzMzMzNDQ0NAo="
}
```

### Encrypt
```bash
http --json POST http://localhost:8200/v1/crypto-srv/encrypt/mysecretkey X-Vault-Token:$VAULT_TOKEN < payloads/to-encrypt-data.json 
```
`Response`
```json
{
    "auth": null,
    "data": {
        "ciphertext": "vault:v1:MQzV+egvco/guzAZ+QPhhnwMeDmOpAukniGEvGL00VVFVYF3J/2u97GjrGP8",
        "key_version": 1
    },
    "lease_duration": 0,
    "lease_id": "",
    "renewable": false,
    "request_id": "57f90884-711d-2964-1df7-722a4fd801f3",
    "warnings": null,
    "wrap_info": null
}
```

### Decrypt
```bash
http --json POST http://localhost:8200/v1/crypto-srv/decrypt/mysecretkey X-Vault-Token:$VAULT_TOKEN < payloads/to-dencrypt-data.json
```

`payload`
```json
{
  "ciphertext": "vault:v1:MQzV+egvco/guzAZ+QPhhnwMeDmOpAukniGEvGL00VVFVYF3J/2u97GjrGP8"
}
```

`Response`
```json
{
    "auth": null,
    "data": {
        "plaintext": "MTExMTIyMjIzMzMzNDQ0NAo="
    },
    "lease_duration": 0,
    "lease_id": "",
    "renewable": false,
    "request_id": "ebb9e588-9cda-a40c-c275-aec9f1c5b598",
    "warnings": null,
    "wrap_info": null
}
```

`NOTA`: texto plano encodado em base64.

```bash
echo "MTExMTIyMjIzMzMzNDQ0NAo=" | base64 -d
```

---
## References

* [TestDriven IO](https://testdriven.io/blog/managing-secrets-with-vault-and-consul/)
* [Vault Storage](https://www.vaultproject.io/docs/configuration/storage/index.html)
* [Vault Secret](https://www.vaultproject.io/docs/secrets/index.html)
* [Vault Auth](https://www.vaultproject.io/docs/auth/index.html)
* [Vault Audit](https://www.vaultproject.io/docs/auth/index.html)
* [Vault Listener](https://www.vaultproject.io/docs/configuration/listener/tcp.html)
* [Vault UI](https://www.vaultproject.io/docs/configuration/ui/index.html)
* [Vault Audit](https://www.vaultproject.io/docs/audit/index.html)
* [Vault Static Secret](https://www.vaultproject.io/guides/secret-mgmt/static-secrets.html)
* [Vault Dynamic Secret](https://www.vaultproject.io/intro/getting-started/dynamic-secrets.html)
* [Why we need Dynamic Secrets](https://www.hashicorp.com/blog/why-we-need-dynamic-secrets)
* [Vault Policies](https://www.vaultproject.io/docs/concepts/policies.html#root-policy)
* [Shamir's Secret Sharing](https://en.wikipedia.org/wiki/Shamir's_Secret_Sharing)