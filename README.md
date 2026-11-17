# seenit-infra

Infrastruktur og deploy-opsætning for SeenIt.

SeenIt kører på én Hetzner Cloud-server, hvor Docker Compose starter fire containere:

| Container | Rolle |
|---|---|
| `caddy` | Reverse proxy og automatisk HTTPS (Let's Encrypt) |
| `web` | Next.js-frontend (image fra GHCR) |
| `backend` | NestJS-API med TypeORM (image fra GHCR) |
| `postgres` | Database med data i et Docker volume |

Images til `web` og `backend` bygges i GitHub Actions i hvert sit repo og pushes til GitHub Container Registry. Dette repo bygger ingen kode, men henter de færdige images.

## Filer

- `docker-compose.yml`: de fire containere og deres netværk
- `Caddyfile`: routing fra domæne til `web` og fra `api.`-subdomæne til `backend`
- `.env.example`: skabelon til miljøvariabler (den rigtige `.env` ligger kun på serveren)

## Domæne

Vi bruger sslip.io, så der skal ikke oprettes DNS-records:

- Frontend: `https://<ip-med-bindestreger>.sslip.io`
- API: `https://api.<ip-med-bindestreger>.sslip.io`

## Første deploy

```bash
ssh root@<server-ip>
cd /opt
git clone git@github.com:<organisation>/seenit-infra.git
cd seenit-infra
cp .env.example .env && nano .env
chmod 600 .env
docker compose up -d
```

## Opdatering

Ændringer laves lokalt og pushes til GitHub. På serveren:

```bash
cd /opt/seenit-infra
git pull
docker compose pull
docker compose up -d
```

Den eneste fil, der redigeres direkte på serveren, er `.env`.

## Relaterede repos

- `seenit-web-frontend`: Next.js
- `seenit-backend`: NestJS
- `seenit-mobile`: React Native (Expo)
