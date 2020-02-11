# Hashicorp Vault Setup with Docker

## Start, initialise and unseal vault server

```
# [1] start vault server in docker container
docker-compose up -d

# [2] initialise vault server either:

# run following to exec into container from host:
docker exec -it vault-server /bin/sh
# then in container:
vault operator init

# OR run the following from host:
docker exec vault-server vault operator init

# [3] take note of unseal keys and root token

# [4] unseal (done either in container or from host)
vault operator unseal
vault operator unseal either_key_is_here
vault operator unseal # or key is entered at prompt

# Note:
# - the 'vault operator unseal' command needs to be entered 3 times
# - either the key is provided in the cli or at a prompt which is provided if key is not
# - using the prompt is the recommended approach to avoid keys remaining in terminal history
```

Vault server is now started, initialised and unsealed. It will remain sealed until the server is either restarted or resealed through API.

## Interact with vault server

Before a Vault client can interact with Vault, it must authenticate against one of the auth methods (GitHub, LDAP, AppRole,etc.). Authentication works by verifying our identity and then generating a token to associate with that identity.

The following example uses the root token

```
vault login root_token_here
```

## Enable KV (key value) secrets engine

```
# enable secrets engine kv v1.
# (see https://www.vaultproject.io/docs/secrets/kv/ for differences between versions)
vault secrets enable -version=1 -path=secrets kv
```

List secret engines with:

```
vault secrets list --detailed
```

## Basic storage and retrieval with API

### Store

The general command form to store a secret is:

```
vault storageEngine put pathToSecret secret
```

The following example uses the kv (key-value) store

```
vault kv put secrets/hello foo=world
```

### Retrieve

The general command form to retrieve a secret is:

```
vault storageEngine get pathToSecret
```

The following example uses the kv (key-value) store

```
vault kv get secrets/hello
```

## Notes

### Restarting

- the file directory holds vault, so restarting server will seal this vault. So when restarting either unseal or delete directory and start from scratch
- a useful cli command to do a complete restart:

```
echo "bring down old vault server" && \
docker-compose down && \
echo "remove old vault filesystem store" && \
rm -rf ./vault/file && \
echo "bring up new vault server" && \
docker-compose up -d && \
echo "wait for server to start ..." && \
sleep 5 && \
echo "initialise new vault server" && \
docker exec vault-server vault operator init
```

## References:

- https://www.bogotobogo.com/DevOps/Docker/Docker-Vault-Consul.php
- https://learn.hashicorp.com/vault/getting-started
- https://hub.docker.com/_/vault
- https://www.vaultproject.io/docs/
