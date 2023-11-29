# Containerising Kong

This repo contains a [Docker Compose file](docker-compose.yaml) which defines three services to allow Kong Gateway to run in a Docker container.

## Services

### Postgres Container

The postgres service defines a container based on the [postgres:13](https://hub.docker.com/layers/library/postgres/13/images/sha256-88dbeb451d370c97d8a19a58d550a6ab62048faa0f1a3187a9e265317241d2e6?context=explore) image. This is used to persist data about Services, Routes and Consumers.

#### Postgres Service Attributes

- `restart` set to `always` - the container will always restart if stopped.
- `POSTGRES_USER` and `POSTGRES_DB` environment variables set to `kong` - the default value that Kong Gateway expects.
- `POSTGRES_PASSWORD` set to any string for the database password.
- Port `5432` on the container is mapped to the host's port `5432` which can be can be accesed by other services.

### Kong Bootstrap Container

This service is based on the [Kong official image](https://hub.docker.com/_/kong). Used to configure the postgres database through the command:  

```bash
kong migrations bootstrap
```

#### Bootstrap Service Attributes

- This service `depends_on` the `postgres` service - this service will start up after the `postgres` service.
- `KONG_DATABASE` set to `postgres` - specifies the type of database Kong is using.
- `KONG_PG_HOST` set to `kong-databse` - the name of the Postgres Docker container to be configured.
- `KONG_PG_PASSWORD` set to `kongpass` - the `POSTGRES_PASSWORD` set in the `postgres service`.
- `restart` set to `on-failure` - this service is only required to be run once and thus only needs to restart if it fails.

### Kong

This service is also based on the [Kong official image](https://hub.docker.com/_/kong) and will be the container which runs the Kong Gateway.

#### Kong Service Attributes

- `restart` set to `always` - the container will always restart if stopped.
- This service `depends_on` the `kong-bootstrap` service - this service will start up after the `kong-bootstrap` service.
- `KONG_DATABASE` set to `postgres` - specifies the type of database Kong is using.
- `KONG_PG_HOST` set to `kong-databse` - the name of the Postgres Docker container to be configured.
- `KONG_PG_USER` set to `kong` - the `POSTGRES_USER` set in the `postgres` service.
- `KONG_PG_PASSWORD` set to `kongpass` - the `POSTGRES_PASSWORD` set in the `postgres` service.
- All `_LOG` variables set filepaths for the logs to output to.
- `KONG_ADMIN_LISTEN` set to `0.0.0.0:8001` - specifies the port the Kong Admin API listens on for requests.
- `KONG_ADMIN_GUI_URL` set to `http://localhost:8002` - specifies the URL for accessing the Kong Manager GUI.
- Ports mappings `8000:8000` and `8443:8443` from the host's ports to container's ports - which take requests from Consumers through in HTTP and HTTPS, respectively.
- Ports mappings `8001:8001` and `8444:8444` from the host's ports to container's ports - which listen for requests to the Admin API through HTTP and HTTPS, respectively.
- Port mapping `8002:8002` maps the host's `8002` port to the container's `8002` port - which is used to listen for HTTP requests for the Kong Manager GUI.

## Running the Services

Start the Kong Gateway:

```bash
docker compose up -d
```

Access the `/services` endpoint to confirm the whether the Admin API is available:

```bash
curl -i -X GET --url http://localhost:8001/services
```

Access the Kong Manager GUI:

```txt
http://localhost:8002
```

You should receive a response with a status code of `200`.
