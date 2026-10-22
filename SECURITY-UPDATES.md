# Container security updates

The October 2026 eMudhra remediation updates Ghost, Caddy, Cloudflare Tunnel,
ActivityPub and analytics. The ActivityPub application and migration images must
use the same release. ActivityPub and analytics still bundle older Node runtimes,
so `docker/Dockerfile.security` upgrades Node within each image's existing major
version and installs the patched Alpine certificate package. The backup uses the
certificate-only target.

Back up MySQL and the Ghost content volume before upgrading. Pull the images,
build the companion images, then recreate the selected services. Avoid starting
`tinybird-deploy` as part of routine security updates: it changes the remote
Tinybird workspace.

```sh
docker compose --profile analytics --profile activitypub --profile backup pull ghost caddy cloudflared activitypub-migrate
docker compose --profile analytics --profile activitypub --profile backup build --pull activitypub traffic-analytics backup
docker compose --profile activitypub up -d --no-deps activitypub-migrate
docker compose --profile analytics --profile activitypub --profile backup up -d --no-deps ghost caddy cloudflared activitypub traffic-analytics backup
```

Wait for `activitypub-migrate` to exit successfully before recreating ActivityPub.
Verify service health and that both the Node HTTPS client and `fetch` reject the
[eMudhra test endpoints](https://emudhra.we-have-a-problem.com/).
A host-level certificate check does not verify container trust stores.
