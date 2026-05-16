# EKG.Test.E2E — End-to-End Test Environments

Docker Compose environments used for local integration testing and manual QA of EKG service stacks. Each subfolder is a self-contained environment that can be started with a single `docker compose up` command.

---

## Environments

### AdminJpe (`AdminJpe/`)

Full stack for testing the Admin + JPE Backoffice flow. Starts all infrastructure and services needed to exercise:

- Admin user registration and login (`EKG.App.Admin`)
- Company/domain management
- Jackpot configuration and lifecycle via JPE Admin API (`EKG.App.Jpe` / `EKG.App.Jpe.Admin`)
- Jackpot Engine Backoffice UI embedded in Admin (`EKG.App.Jpe.BO`)

#### Services

| Service | Image | Port | Description |
|---|---|---|---|
| `mongo` | `mongo:7` | 27017 | MongoDB — shared by all services |
| `redis` | `redis:7-alpine` | 6379 | Redis — sessions, caches, leader election |
| `rabbitmq` | `rabbitmq:3-management` | 5672 / 15672 | RabbitMQ — jackpot event bus |
| `pam` | `ekg.app.pam:latest` | 5030 | Player account manager |
| `wallet` | `ekg.app.wallet:latest` | 5020 | Player wallet |
| `jpe-admin` | `ekg.app.jpe.admin:latest` | 5010 | JPE Admin API (jackpot CRUD) |
| `jpe-be` | `ekg.app.jpe.be:latest` | 5011 | JPE game event processor |
| `jpe-bo` | `ekg.app.jpe.bo:latest` | 5090 | JPE Backoffice UI |
| `admin-be` | `ekg.app.admin.be:latest` | 5080 | Admin REST API |
| `admin-ui` | `ekg.app.admin.ui:latest` | 5071 | Admin frontend |

#### Quick start

```bash
cd AdminJpe

# Pull latest images (all services)
docker compose pull

# Start everything
docker compose up -d

# Tail logs
docker compose logs -f
```

All images are pulled from `ghcr.io/evgenkovalenko/`.

#### Access points

| URL | Service |
|---|---|
| http://localhost:5071 | Admin UI — login with `admin` / `Admin1234!` |
| http://localhost:5090 | JPE Backoffice (also embedded in Admin UI) |
| http://localhost:5080/scalar/v1 | Admin BE API docs |
| http://localhost:5010 | JPE Admin API |
| http://localhost:15672 | RabbitMQ management UI — `guest` / `guest` |

#### Default seed credentials

| Field | Value |
|---|---|
| Admin username | `admin` |
| Admin email | `admin@ekg.local` |
| Admin password | `Admin1234!` |

Seeded automatically by `admin-be` on first startup.

#### Key environment variables

`jpe-be` requires RabbitMQ topic configuration for both consumers:

```yaml
MessageBroker__Topics__JackpotDataUpdatesConsumer__QueueName: jpe-jackpot-data-updates
MessageBroker__Topics__JpeGeneralSettingsUpdatedConsumer__QueueName: jpe-general-settings-updates
```

`jpe-bo` requires the Cloudflare R2 widgets URL for skin preview:

```yaml
JpeSettings__WidgetsBaseUrl: https://pub-d441b9842695419293a2ee0db0538a6a.r2.dev
```

---

## Updating images

To pull and restart a single service after a new image is published:

```bash
docker compose pull <service>
docker compose up -d --force-recreate <service>
```

Example — update the JPE BO after a new build:

```bash
docker compose pull jpe-bo
docker compose up -d --force-recreate jpe-bo
```

---

## Tiltfile

Each environment also has a `Tiltfile` for use with [Tilt](https://tilt.dev/). Tilt watches for image changes and auto-restarts services during active development.

```bash
cd AdminJpe
tilt up
```

See `Tiltfile` and `Tiltfile.cloud` in each environment folder for configuration details.
