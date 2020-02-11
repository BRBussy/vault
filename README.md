# Hashicorp Vault

```
# to start vault server in docker container
docker-compose up -d


# to initialise vault server either:

# [1a] run following to exec into container from host:
docker exec -it vault-server /bin/sh
# [1b] then in container:
vault operator init

# OR

# [2]
docker exec vault-server vault operator init
```
