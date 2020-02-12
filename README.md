# Hashicorp Vault Docker - setup and basic use

## Setup

### Start and initialise vault server

Run the following from the root of this repository:

```
echo "start vault server in docker container" && \
docker-compose up -d && \

echo "initialise vault server" && \
docker exec vault-server vault operator init
```

**NB: Take note of the unseal keys and root token printed out after vault server initialisation! They are required to unseal and login.**

### Unseal Vault

Run the 'vault operator unseal' operation 3 times to unseal. The unseal command can either be entered along with an unseal key on the same line or on its own which will bring up a prompt to enter a key. Examples of running the command:

- Unseal from the host using 'docker exec':

```
docker exec vault-server vault operator unseal __unseal-key-here___  # run command 3x
```

Note: unsealing from host like this it is not possible to use prompt mode (unless docker -it flag is used).

- Unseal from within the container:

```
vault operator unseal                                # 1/3
vault operator unseal either_key_is_here             # 2/3
vault operator unseal # or key is entered at prompt  # 3/3
```

**Note: Unsealing from within the container and entering keys at prompt is recommended to prevent unseal keys remaining in terminal history.**

Vault server is now started, initialised and unsealed. It will remain unsealed until the server is either stopped, restarted or resealed through API.

Note that if the server comes down in any way (i.e. if the container is removed) the sealed vault remains in the filesystem located at 'vault/file'. If it is then started again it will not be possbile to re-initialse the server unless these files are removed.

---

## Interact with vault server

Before a Vault client can interact with Vault, it must authenticate against one of the auth methods (GitHub, LDAP, AppRole,etc.). Authentication works by verifying our identity and then generating a token to associate with that identity.

The following example uses the root token

```
vault login root_token_here
```

## Secret Engines

Vault behaves as a virtual file system. A secrets engine is the backend 'plugin' to which read/write/delete/list operations for this 'file system' are sent. The chosen secrets engine decides how to implement these operations. Secrets engines need to be enabled at particular path. Following is a basic example setting up the KV secrets engine. (https://www.vaultproject.io/docs/secrets/)

### Enable KV (key value) secrets engine

The kv secrets engine is used to store arbitrary secrets within the configured physical storage for Vault. This backend can be run in one of two modes. It can be a generic Key-Value store (v1) that stores one value for a key or versioning can be enabled (v2) and a configurable number of versions for each key will be stored. (https://www.vaultproject.io/docs/secrets/kv/)

```
# enable secrets engine kv v1.
vault secrets enable -version=1 -path=secrets kv
```

List secrets engines and the paths on which they are mounted with:

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

List secrets in kv engine with:

```
vault kv list engineMountPath
```

The following example uses the kv (key-value) store

```
vault kv get secrets/hello
```

## Notes

### Restarting

- the file directory holds vault, so restarting server will seal this vault. So when restarting either unseal or delete directory and start from scratch
- a useful cli command to do a complete restart (run from root of this repo):

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
