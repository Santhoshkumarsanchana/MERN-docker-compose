# A simple MERN stack application 

### Create a network for the docker containers

`docker network create demo`

### Build the client 

```sh
cd mern/frontend
docker build -t mern-frontend .
```

### Run the client

`docker run --name=frontend --network=demo -d -p 5173:5173 mern-frontend`

### Verify the client is running

Open your browser and type `http://localhost:5173`

### Run the mongodb container

`docker run --network=demo --name mongodb -d -p 27017:27017 -v ~/opt/data:/data/db mongo:latest`

### Build the server

```sh
cd mern/backend
docker build -t mern-backend .
```

### Run the server

`docker run --name=backend --network=demo -d -p 5050:5050 mern-backend`

## Using Docker Compose

`docker compose up -d`

## Walkthrough

### 01. Preparing EC2: opening ports 5173, 5050 and 27017
![](screenshots/01-security-group-ports.png)
Wide-open rules were used for this lab only.

### 02. Frontend Dockerfile (initial version)
![](screenshots/02-frontend-dockerfile.png)
Later changed to `node:18-alpine`.

### 03. Frontend container running (no backend yet)
![](screenshots/03-frontend-empty.png)

### 04. MongoDB fails: no space left on device
![](screenshots/04-mongo-no-space-error.png)

### 05. Creating a custom Docker network (`mern`)
![](screenshots/05-docker-network-mern.png)

### 06. Fixing permissions for the MongoDB data volume
![](screenshots/06-fix-volume-permissions.png)

### 07. Root cause: `mongo:latest` incompatible with kernel 6.19+
![](screenshots/07-mongo-kernel-incompatibility.png)

### 08. MongoDB running (pinned `mongo:7.0.12`)
![](screenshots/08-mongodb-running-27017.png)

### 09. Backend API running on port 5050
![](screenshots/09-backend-running-5050.png)
`Cannot GET /` is expected: the API only serves `/record`.

### 10. Frontend build fails: disk full
![](screenshots/10-build-no-space-error.png)

### 11. Still failing on `node:18-alpine`: Cypress download fills the disk
![](screenshots/11-alpine-build-still-no-space.png)

### 12. Frontend cannot reach the backend (DevTools)
![](screenshots/12-frontend-fetch-failed.png)

### 13. Cause: `localhost:5050` in `Record.jsx`
![](screenshots/13-localhost-api-url-issue.png)
`localhost` points to my own machine, not the EC2 server. Fix: use the EC2 public IP.

### 14. Rebuild with `node:18-alpine`; backend connected to MongoDB
![](screenshots/14-rebuild-backend-connected.png)

### 15. Final working MERN app on EC2
![](screenshots/15-final-app-working.png)

## Issues & fixes
| Problem | Cause | Fix |
|---|---|---|
| `no space left on device` | Small disk + large images (Cypress) | Pruned images, `node:18-alpine`, freed disk space |
| MongoDB exits immediately | `mongo:latest` incompatible with kernel 6.19+ | Pinned `mongo:7.0.12` |
| `mkdir: Permission denied` | `~/opt` owned by root | `sudo chown -R ubuntu:ubuntu ~/opt` |
| Empty table, failed fetches | `localhost:5050` hard-coded in `Record.jsx` | Use EC2 public IP, open port 5050 |
