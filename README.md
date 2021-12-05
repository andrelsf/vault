# Vault

Learning about Vault Hashcoorp

## Init Environment

```bash
docker-compose up -d
docker-compose exec vault /bin/sh
```

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

vault secrets enable -version=2 kv
vault kv enable-versioning kv
vault secrets list
```

### Enable Auditing

- [Vault Audit](https://www.vaultproject.io/docs/audit)

```bash
vault audit enable -description=Auditlogs file file_path=/vault/logs/audit.logs
vault audit list
```

### Key-Value Engine

```bash
# create or update a value for a key
vault kv put kv/mysecrets key=VAIII
vault kv get kv/mysecrets
vault kv metadata get kv/mysecrets
vault kv put kv/mysecrets key=FUIII
vault kv metadata get kv/mysecrets
vault kv get -version=1 kv/mysecrets
vault kv get -version=2 kv/mysecrets
vault kv delete -versions=1 kv/mysecrets
vault kv metadata get kv/mysecrets
vault kv undelete -versions=1 kv/mysecrets
vault kv get -version=1 kv/mysecrets
vault kv destroy -versions=1 kv/mysecrets
vault kv put kv/mysecrets email=test.email@gmail.com password=123456789
vault kv get -field email kv/mysecrets
vault kv get -field password kv/mysecrets

vault secrets enable -path=multicode -description="KV Secrets for project multicode" -version=2 kv
vault kv enable-versioning multicode

# Forma de destruir versions
vault kv destroy -versions=1,2,3 multicode/secrets
```

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

# Autenticando o usuario
vault login -method=userpass -path=multicode username=andrelsf password=multicode

# Autenticando via TOKEN
vault login token=<USER_TOKEN>

# Removendo usuário
vault delete auth/multicode/users/andrelsf
```

`Nota:` Neste processo o Vault irá gerar um token que pode ser usado para autenticar no via TOKEN.

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


```hcl
path "multicode/data/secrets" {
    capabilities = [ "read" ]
}
```

```bash
vault policy list
vault policy write multicode-ro-policy multicode-ro-policy.hcl
vault write auth/multicode/users/andrelsf password=multicode policies=multicode-ro-policy
vault login -method=userpass -path=multicode username=andrelsf password=multicode
vault kv get multicode/secrets
vault kv put multicode/secrets key=UPDATED_VALUE
```

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