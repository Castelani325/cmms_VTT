---
name: atlas-cmms-selfhosted-deploy
description: How the Atlas CMMS instance is deployed self-hosted on the Venttos server and how license limits were removed
metadata:
  type: project
---

Atlas CMMS roda self-hosted no servidor Ubuntu `192.168.3.26` (SSH `engineering@`, Docker exige `sudo`), em `~/projects/Atlas_CMMS`. Esse diretório do servidor NÃO é um clone git — é cópia parcial (`docker-compose.yml`, `frontend/`, `api/` quando enviado, `config/`, `logo/`, e o `.env` com os segredos reais). Portas: frontend `8096` (servido sob `/Atlas`), API `8097`, MinIO `9000/9001`, Postgres `5432` (db `atlas`, container `atlas_db`).

Deploy do backend é HÍBRIDO: originalmente o serviço `api` usava `image: intelloop/atlas-cmms-backend` (upstream da Grash, prebuilt). Para rodar código modificado, mudamos o serviço `api` no `docker-compose.yml` do servidor para `build: { context: ./api, dockerfile: Dockerfile }` + `image: atlas-cmms-backend-custom`. Ciclo de deploy do backend: `scp -r api engineering@192.168.3.26:~/projects/Atlas_CMMS/` (do PC) → `sudo docker compose build api` → `sudo docker compose up -d api`. O `api/Dockerfile` é multi-stage Maven, auto-contido. O frontend já era `build: ./frontend` (imagem `atlas-cmms-frontend-custom`).

Limites de licença REMOVIDOS: o projeto é AGPL-3.0 (modificação para uso interno é permitida). Toda limitação (5 usuários, 50 assets, etc.) e o crash de boot vinham de `checkUsageBasedLimit` guardado por `licenseService.hasEntitlement(...)`. Neutralizado num único ponto: `api/.../service/LicenseService.java` `getLicensingState()` agora retorna sempre `valid=true` com TODOS os `LicenseEntitlement.values()` como entitlements. Isso destrava backend E frontend (que lê `/license/state`). Código de validação Keygen ficou inerte mas intacto (reversível). Sem `-Werror` no pom, métodos privados órfãos não quebram o build.

Cuidado conhecido: arquivos editados no Windows e enviados por `scp` podem ter CRLF, o que faz `sed ...$` não casar no servidor — rodar `sed -i 's/\r$//' arquivo` antes. Ver [[atlas-cmms-superadmin-and-login]].
