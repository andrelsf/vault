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
vault audit enable file file_path=/vault/logs/audit.logs
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