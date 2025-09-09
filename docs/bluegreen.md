Blue/Green deployment runs two stacks side-by-side: one live, one idle. You
deploy to the idle stack, test it, and when ready, swap roles — giving
near-zero downtime and easy rollback.

![Blue/Green](assets/bluegreen.png)

## 1. Adjust the Compose file

Remove the Caddy `ports:` section in `compose.yaml`. Instead of exposing ports
in the stacks, a "front proxy" will expose ports and proxy to the active stack.

Set `CADDY_SITE_ADDRESS` to only `:80` (leaving TLS termination to the front
proxy):

```yaml title="compose.yaml"
caddy:
  environment:
    CADDY_SITE_ADDRESS: :80
```

To share data between the two stacks (database, uploads, etc.), give volumes
explicit names:

```yaml title="compose.yaml"
volumes:
  postgres_data:
    name: postgres-data
  user_data:
    name: user-data
```

## 2. Add a Front Proxy

The _front proxy_ is a single container that binds ports `80` and `443` on the
server and routes requests into either the Blue or Green stack.

On the server, create a simple `Caddyfile`:

```caddyfile title="Caddyfile"
api.myapp.com {
  reverse_proxy blue_caddy:80
}

# Optionally point a second hostname to the idle stack for testing
next.myapp.com {
  reverse_proxy green_caddy:80
}
```

The front proxy manages TLS, so give it a persistent volume for certificates:

```sh
docker volume create caddy_data
```

Create networks for the two stacks:

```sh
docker network create blue_default
docker network create green_default
```

Start the proxy and attach it to both networks:

```sh
docker run -d \
  --name front-proxy \
  -p 80:80 -p 443:443 \
  -v ./Caddyfile:/etc/caddy/Caddyfile \
  -v caddy_data:/data \
  --network blue_default \
  --network green_default \
  caddy:2
```

## 3. Deploying/Upgrading

Deploying is the same as [before](deploying.md), but now we're deploying the
_idle stack_. For this example, `green` is idle so that's the one we're
deploying.

Create `blue` and `green` directories on the server and deploy `compose.yaml`
into the idle stack's directory:

```sh
scp compose.yaml youruser@yourserver:green/compose.yaml
```

Shell into the server and bring up the idle stack:

```sh
cd green
docker compose pull
docker compose up -d
```

Docker will use the directory name `green` as the project name, creating
different containers, volumes and networks than the `blue` stack.

### Flip traffic

Point traffic to the `green` stack, and make `blue` idle:

```caddyfile title="Caddyfile"
api.myapp.com {
  reverse_proxy green_caddy:80
}

next.myapp.com {
  reverse_proxy blue_caddy:80
}
```

Reload the front proxy's config:

```sh
docker exec front-proxy caddy reload
```

Cutover is instant. Green is now live, and Blue is the idle stack.

And rollback is simple: flip the `Caddyfile` back and `caddy reload` again.
