# Hashicorp Vault

References:

- https://www.bogotobogo.com/DevOps/Docker/Docker-Vault-Consul.php
- https://learn.hashicorp.com/vault/getting-started

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

# [4] unseal (again the following can be done either in container or from host)


```
