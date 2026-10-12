# Udagram Microservices on Kubernetes

Udagram is a photo sharing application. This repo takes a monolithic Node.js API and splits it into independently deployable microservices, each packaged as a Docker container, built by a CI pipeline, and deployed to a Kubernetes cluster behind an nginx reverse proxy.

Built in 2020 during Udacity's Cloud Developer nanodegree.

## Architecture

```
                ┌──────────────┐
  Browser ────▶ │   Frontend   │  Ionic / Angular
                └──────┬───────┘
                       │ /api/v0
                ┌──────▼───────┐
                │ Reverse proxy│  nginx
                └───┬──────┬───┘
                    │      │
          ┌─────────▼┐   ┌─▼─────────┐
          │  Users   │   │   Feed    │  Node.js / Express / TypeScript
          └────┬─────┘   └─────┬─────┘
               │               │
          ┌────▼───────────────▼────┐      ┌────────┐
          │      PostgreSQL         │      │ AWS S3 │  image storage
          └─────────────────────────┘      └────────┘
```

| Service | Folder | Responsibility |
|---|---|---|
| Frontend | `frontend/` | Ionic / Angular web client |
| Users | `users/` | Registration, login, JWT issuing and verification, user lookup |
| Feed | `feed/` | Feed items, PostgreSQL via Sequelize, presigned S3 URLs for image upload |
| Reverse proxy | `reverseproxy/` | nginx routing `/api/v0/users` and `/api/v0/feed` to the right service |

## Tech
* Node.js, Express and TypeScript
* Angular / Ionic frontend
* PostgreSQL with Sequelize ORM
* AWS S3 with presigned URLs
* JWT authentication with bcrypt password hashing
* Docker, one image per service
* Kubernetes deployments and services, with a ConfigMap and secrets for configuration
* Horizontal Pod Autoscaler on the feed service, which runs with multiple replicas
* Travis CI building every image and pushing to Docker Hub

## API

**Users service**
| Method | Path | Description |
|---|---|---|
| POST | /api/v0/users/auth | Register a new user |
| POST | /api/v0/users/auth/login | Log in and receive a JWT |
| GET | /api/v0/users/auth/verification | Verify a JWT |
| GET | /api/v0/users/:id | Get a user |

**Feed service**
| Method | Path | Description |
|---|---|---|
| GET | /api/v0/feed | List feed items |
| GET | /api/v0/feed/:id | Get a feed item |
| GET | /api/v0/feed/signed-url/:fileName | Get a presigned S3 upload URL (authenticated) |
| POST | /api/v0/feed | Create a feed item (authenticated) |

## CI/CD

On every push, Travis CI (`.travis.yml`) builds the four Docker images and pushes them to Docker Hub. Kubernetes manifests in each service folder (`deployment.yaml`, `service.yaml`) deploy the images to the cluster. Credentials are supplied through Kubernetes secrets and are not stored in the images.

## Deployment screenshots

The `screenshots/` folder shows the running system:
* [Successful Travis CI build](screenshots/TravisCI-Successful-Build-Jobs.png)
* [Images on Docker Hub](screenshots/DockerHub-Docker-Images.png)
* [Pods running in Kubernetes](screenshots/kubectl%20get%20pods.png)
* [Kubernetes services](screenshots/kubectl%20describe%20services.png)
* [Horizontal Pod Autoscaler](screenshots/kubectl%20describe%20hpa.png)
* [Pod does not expose credentials](screenshots/kubectl%20describe%20pod-does-not-expose-credentials.png)

## Running locally

Each service has its own `package.json` and `Dockerfile`. Set the PostgreSQL, AWS and JWT environment variables, then run `npm install` and `npm run dev` in a service folder, or build its image with `docker build`.

## Note

Uses 2020 era package versions (Node 13, Angular / Ionic of that time), so dependencies would need updating before deploying today.
