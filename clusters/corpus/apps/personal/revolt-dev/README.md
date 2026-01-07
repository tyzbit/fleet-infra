# Stoat (formerly Revolt)

Stoat (formerly Revolt) is a community-oriented chat app with the official instance at
https://stoat.chat.

THIS IS NOT ASSOCIATED WITH REVOLT IN ANY WAY, OFFICIAL OR UNOFFICIAL.
USE AT YOUR OWN RISK.

Borrowed heavily from https://github.com/stoatchat/self-hosted/ with gratitude.

Revolt is referenced many times and I may not rename things like PVCs due to it not being worth the effort to me.

## Architecture

### Revolt services

- Web: Web frontend built with yarn (this imlementation does a find&replace for
  hardcoded values). Backends are Mongo, Redis and RabbitMQ
- Delta(API): Standard API
- File-server: User content, uploads. Backend is Minio/S3
- Proxy: Proxy service - embeds, resizing
- ~~Vortex: Voice. Being rewritten~~
- LiveKit - The new voice & multimedia service (still self-hostable). Not operational for self-hosters as far as I know
- (Events): Realtime events via websocket
- Gifbox: Relatively new, appears to be a proxy specifically for gifs
- Crond: Periodic tasks
- Pushd: Push notifications

### Standard backends

- Mongo: Standard database usage. Used by the API
- Minio: Asset storage (emojis, uploads etc). Used by File-server
- Redis: Session tracking. Used by API
- RabbitMQ: User interactions - mentions, friend requests etc. Used by API

## Prerequisites

#### UPDATE VARIABLES

- Throughout this folder there are references to variables. I use SOPS to
  resolve them, but you may need to do something different. Also some of the variable names are pretty specific to me, so you'll wanna update those anyway.
- Of particular note is `${home_ip}`. Yes, that must be your external IP for voice chat purposes. It might be possible to use STUN/TURN servers but that has not been implemented here.

#### UPDATE CONFIG

- `livekit.yaml` needs to be reviewed
- The config, located at `configmap.yaml`, is opinionated and customized.
- Opinionated things you'll want to consider changing/checking:
  - `[hosts]`
  - `[hosts.livekit]`
  - `[api.livekit]`
  - `[database]`
  - `[rabbit]`
  - `[api.registration]`
    - `invite_only`
    - If enabled, use the commands in the `invite_mongo_query` in the mongodb container to create an initial invite so you can create the first account.
  - `[api.smtp]`
  - `[api.security]`
    - `voso_legacy_token`: can be anything, I just generated a UUID with `uuidgen`.
  - `[api.security.captcha]`
    - You will need to register with hCaptcha and create new keys.
  - `[pushd.vapid]`
    - Use [this script](https://github.com/revoltchat/self-hosted/blob/master/generate_config.sh) to generate these and the file encryption key
  - `[files]`
    - `encryption_key` (see directly above)
  - `[files.s3]`
    - `secret_access_key`: same as $MINIO_ROOT_PASSWORD in the HelmRelease
  - `[features.limits]`
    - Change these to your liking, but **ALSO UPDATE THE COMMAND FOR THE WEB CONTAINER** to match these, because the limits are set client-side AND server-side.
  - Minor thing: `trust_cloudflare = false` seems to work even though I'm behind CloudFlare so I just left it off. It's possible that causes issues I didn't identify though.

#### PORT FORWARD

- The production version uses ports 12301-12320 (UDP) for voice. You'll need
  to forward that to your vortex service however you like (such as by using
  a LoadBalancer-type service)

#### DNS

- I use external-dns, so the DNS defined in the ingresses gets created in my CloudFlare account automatically, but you might have a different setup.

#### RESOURCE LIMITS

- Go through the HelmRelease and adjust limits, maybe a little while after first deployment and users.

#### PERSISTENCE

- I like creating claims separate from HelmReleases and then simply referencing them in the HelmRelease so I can delete HelmReleases and be confident that the volumes won't get deleted, so if you prefer a different pattern, update this.
- The sizes are still initial guesses, and your needs will vary widely based on usage.

#### SERVICES

- I use ClusterIPs because I have BGP to route from my internal network into k8s, so you might want to use NodePort or LoadBalancer here.
- The vortex ports are set in the environment variables for vortex and the ports in the service should match. You can make them whatever you want, and you can have more than 20, I just didn't want to clutter up the HelmRelease and I think my instance won't exceed 20 users connected to voice for a long time.

#### INGRESS

- This will almost definitely need changing, unless you use `ingress-nginx` and `external-dns` like I do. The key parts are rewriting the request for the backend services by chopping off the first stub. You might have to re-implement this in Traefik, ALB Ingress Controller or whatever you use.
  - Example: `/api/settings` from the client needs to be transformed to a request for `/settings` to the API backend.
- **Paths and microservices are not static, be prepared to adjust or update these over time.**

#### TLS

- I use cert-manager, so TLS defined on an Ingress automatically gets a certificate but if you do something different, make sure that gets set up for this.
