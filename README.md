# Hashicorp Vault Docker - Setup and Basic Use Example/Tutorial

## Setup

### Start and initialise vault server

Run the following from repository root on host:

```
echo "start vault server in docker container" && \
docker-compose up -d && \

echo "give some time for server to start ..." && \
sleep 5 && \

echo "initialise vault server" && \
docker exec vault-server vault operator init
```

**NB: Take note of the unseal keys and root token printed out after vault server initialisation! They are required for unseal and login operations.**

### Unseal Vault

Run the 'vault operator unseal' operation 3 times to unseal. The unseal command can either be entered along with an unseal key on the same line or on its own which will bring up a prompt to enter a key. Examples of running the command:

- Unseal from the host using 'docker exec':

```
docker exec vault-server vault operator unseal __unseal-key-here___  # run command 3x with different unseal keys
```

Note: unsealing from host like this it is not possible to use prompt mode (unless docker -it flag is used).

- Unseal from within the container:

```
# open session in container with: docker exec -it vault-server /bin/sh
vault operator unseal                                # 1/3
vault operator unseal either_key_is_here             # 2/3
vault operator unseal # or key is entered at prompt  # 3/3
```

**Note: Unsealing from within the container and entering keys at prompt is recommended to prevent unseal keys remaining in terminal history.**

Vault server is now started, initialised and unsealed. It will remain unsealed until the server is either stopped, restarted or resealed through API.

Note that if the server comes down in any way (e.g. if the container is removed) the sealed vault remains in the filesystem located at 'vault/file' (as configured in config/config.hcl). If it is then started again it will not be possbile to re-initialse the server unless these files are removed since the vault server will interpret the files as the existing already initialised vault. A useful command to perform a complete restart and reinitialisation of the vault server:

```
# -- run from repostory root on host --

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

---

## Interact with vault server

Before a Vault client can interact with Vault, it must authenticate against one of the auth methods (GitHub, LDAP, AppRole,etc.). Authentication works by verifying our identity and then generating a token to associate with that identity.

The following example uses the root token printed out during initialisation to log the vault api client in.

```
# -- run inside container --

vault login root_token_here
```

---

## Secret Engines

Vault behaves as a virtual file system. A secrets engine is the backend 'plugin' to which read/write/delete/list operations for this 'file system' are sent. The chosen secrets engine decides how to implement these operations. Secrets engines need to be enabled at a particular path. Following is a basic example enabling up the KV secrets engine. (https://www.vaultproject.io/docs/secrets/)

### Enable KV (key value) secrets engine

The kv secrets engine is used to store arbitrary secrets within the configured physical storage for Vault. This backend can be run in one of two modes. It can be a generic Key-Value store that stores one value for a key (i.e. v1) or versioning can be enabled and a configurable number of versions for each key will be stored (i.e. v2). (https://www.vaultproject.io/docs/secrets/kv/)

```
# -- run inside container --

# enable secrets engine kv v1.
vault secrets enable -version=1 -path=secrets kv
```

List secrets engines and the paths on which they are mounted with:

```
vault secrets list --detailed
```

## Basic storage and retrieval with API

### Store

The general command to create a secret:

```
vault secretsEngine put pathToSecret secret
```

Example using the kv engine:

```
vault kv put secrets/hello foo=world
```

### Retrieve

The general command to retrieve a secret:

```
vault secretsEngine get pathToSecret
```

Example using the kv engine:

```
vault kv get secrets/hello
```

### List

The general comand to list secrets:

```
vault secretsEngine list engineMountPath
```

Example using kv engine at /secrets mount path:

```
vault kv list /secrets
```

### Delete

The general command to delete a secret:

```
vault secretsEngine delete pathToSecret
```

Example using kv engine:

```
vault kv delete secrets/hello
```

---

## Policies

Policies in Vault control what a user an access. Some built in policies cannot be removed:

- default: set of common permissions included on all tokens by default
- root: gives a token super admin permissions

Policies are written in HCL or JSON.

## Notes

### Restarting

- the file directory holds vault, so restarting server will seal this vault. So when restarting either unseal or delete directory and start from scratch
- a useful cli command to do a complete restart (run from root of this repo):

## References:

- https://www.bogotobogo.com/DevOps/Docker/Docker-Vault-Consul.php
- https://learn.hashicorp.com/vault
- https://hub.docker.com/_/vault
- https://www.vaultproject.io/docs/
