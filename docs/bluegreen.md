# Blue/Green Deployments

Blue/Green is a deployment method where two complete stacks are running
side-by-side. One is serving production traffic, while the other is idle. You
deploy to the idle stack, test it, and when you’re ready you swap roles — the
idle stack becomes production, and the old production becomes idle. This
provides near-zero downtime and an easy rollback path.

We’ll bring up two stacks, `blue` and `green`, with no ports exposed. A
separate lightweight front proxy binds to `:80` and `:443` and routes traffic
to whichever stack is active.

## 1. Caddyfile

Remove the exposed ports from Caddy by removing the `ports:` section in
`compose.yaml`.

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

## 2. Start two Stacks

On the server, bring up two stacks, `blue` and `green`:

```sh
docker compose -p blue up -d
docker compose -p green up -d
```

## 3. Front Proxy

The front proxy is a single Caddy container that binds `:80` and `:443` on the
server and routes requests into either the Blue or Green stack.

On the server, create a simple `Caddyfile`:

```caddyfile title="Caddyfile"
api.myapp.com reverse_proxy blue_caddy:80

# Optionally point a second hostname to the idle stack for testing
next.myapp.com reverse_proxy blue_caddy:80
```

The front proxy manages TLS, so give it a persistent volume for certificates:

```sh
docker volume create caddy_data
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

## 4. Deploying

Update the idle stack:

```sh
docker compose pull
docker compose -p green up -d
```

Edit the front proxy's config to flip traffic:

```caddyfile title="Caddyfile"
api.myapp.com reverse_proxy green_caddy:80
next.myapp.com reverse_proxy blue_caddy:80
```

Restart Caddy:

```sh
docker exec front-proxy caddy reload
```

Cutover is instant. Green is now live, and Blue is the idle stack.

And rollback is simple: flip the `Caddyfile` back and `caddy reload`.
