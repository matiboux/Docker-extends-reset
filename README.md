# Docker Compose experiments with `extends` and `reset`

```bash
docker compose -f docker-compose.yml -f docker-compose.override.yml config
```


## Base files

`docker-compose-a.yml` sets:
- Service `app`, from scratch
  - Port `80:80` set

`docker-compose-b.yml` sets:
- Service `app`, from scratch
  - Port `80:80` set

`docker-compose.yml` sets:
- Service `a`, extending `app` from `docker-compose-a.yml`
- Service `b`, extending `app` from `docker-compose-b.yml`
  - Ports unset (reset), then port `8080:80` set
- Service `c`, from scratch
  - Port `5000:80` set
  - Environment variable `SECRET` set to `secret`

`docker-compose.override.yml` sets:
- Service `c`, extending `c` from `docker-compose.yml`
  - Ports unset (reset), then port `8081:80` set
  - Environment variable `SECRET` unset (reset) (set to `null`)


## Docker Compose config

### Expected result

When running `docker compose -f docker-compose.yml -f docker-compose.override.yml config`, the expected result is:

```yaml
services:
  a:
    ports:
    - target: 80
      published: 80
  b:
    ports:
    - target: 80
      published: 8080
  c:
    environment:
      SECRET:
    ports:
    - target: 80
      published: 8081
```

### Actual result

However, the actual result is:

```yaml
services:
  a:
    ports:
    - target: 80
      published: 80
  b:
    ports:
    - target: 80
      published: 80
    - target: 80
      published: 8080
  c:
    environment:
      SECRET: 'null'
    ports: []
```

### Issues

There are three issues here:
- Service `b` has two ports, instead of one.
  - The port `80:80` from `app` in `docker-compose-b.yml` has not been unset, even though it should have been.
  - The port `8080:80` from `docker-compose.yml` has been set correctly.
- Service `c` has `SECRET` environment variable set, even though it should have been unset.
  - The value `'null'` overrides the value `'secret'` from `docker-compose.yml`.
  - However, the value should have been unset, not set to the `'null'` string.
- Service `c` has no ports, instead of one.
  - The port `5000:80` from `docker-compose.yml` has been unset correctly.
  - The port `8081:80` from `docker-compose.override.yml` has not been set, even though it should have been.
