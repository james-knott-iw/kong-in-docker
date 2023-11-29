# Containerising Kong

This repo contains a [Docker Compose file](docker-compose.yaml) which defines three services to allow Kong Gateway to run in a Docker container.

## Services

### Postgres Container

The postgres service defines a container based on the [postgres:13](https://hub.docker.com/layers/library/postgres/13/images/sha256-88dbeb451d370c97d8a19a58d550a6ab62048faa0f1a3187a9e265317241d2e6?context=explore) image. This is used to persist data about Services, Routes and Consumers.

#### Attributes

- `restart` set to `always` - the container will always restart if stopped
- `POSTGRES_USER` and `POSTGRES_DB` environment variables set to `kong` - the default value that Kong Gateway expects.
- `POSTGRES_PASSWORD` set to any string for the database password.
- Port `5432` on the container is mapped to the host's port `5432` which can be can be accesed by other services.

## Running the Services

Start the Kong Gateway:

```bash
docker compose up -d
```

Access the `/services` endpoint to confirm the whether the Admin API is available:

```bash
curl -i -X GET --url http://localhost:8001/services
```

You should receive a response with a status code of `200`.
