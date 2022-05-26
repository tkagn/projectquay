# Project Quay

Project Quay consist of several products:

- PostgreSQL container: Database for image metadata
- Redis container: In-memory key/value store
- Clair container: Container vulnerability scanner
- Quay container: Container registry

All products run within the same pod and use the same pod network.

Setup Project Quay working directory for persistence:

```bash
mkdir -p /ProjectQuay/{pgdata,clair,quaydata}

```

## Deploy Postgres
```bash
podman volume create pgdata

podman pod create --name postgres

podman run -d --rm --name postgresql \
    --network quay \
    -e POSTGRES_PASSWORD='podman' \
	-e POSTGRES_DB=quay \
	-p 5432:5432 \
	-v pgdata:/var/lib/postgresql/data:Z \
    -d docker.io/library/postgres
```

Add pg_trgm module
```bash
$ podman exec -it postgresql /bin/bash -c 'echo "CREATE EXTENSION IF NOT EXISTS pg_trgm" | psql -d quay -U postgres'
```

## Deploy/run Redis

`  podman pod create --name redis `

```bash
podman run -d --rm --name redis-container \
        --network quay \
        -p 6379:6379 \
        docker.io/library/redis \
        --requirepass strongpassword
```

# Deploy/run Clair

```
podman run -d --name clair -e GODEBUG=x509ignoreCN=0 -e CLAIR_MODE=combo -e CLAIR_CONF=/config/config.yml --network quay -p 6060-6061:6060-6061 -p 8082:8080 -v clair-config:/config:Z quay.io/projectquay/clair:4.3.5
```


## Deploy/run Quay Config Tool
```bash
podman volume create quaydata

podman pod create --name quay
//////////////////////////////////////////////////////
podman run --rm -d --name quay_config --network quay --pod quay -p 8443:8443 -p 8081:8080 \
   -v quay-config:/conf/stack:z

 quay.io/projectquay/quay:latest config secret
```

## Deploy/run Quay
```bash
podman run --rm -p 8080:8080 -p 8443:8443 \
   --name=quay \
   --network quay \
   --privileged=true \
   -v quay-config:/conf/stack:z \
   -v quaydata:/datastorage:z \
   -d quay.io/projectquay/quay:latest
   ```
