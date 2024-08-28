
---

# OpenELIS Global Docker Compose Setup

This repository contains the Docker Compose configuration for deploying the OpenELIS Global system. It includes services for certificate generation, the PostgreSQL database, the OpenELIS Global web application, an external FHIR API, the frontend, and an NGINX proxy.

## Prerequisites

- Docker (version 19.03 or higher)
- Docker Compose (version 1.27 or higher)

## Services Overview

### 1. **certs**
   - **Image:** `itechuw/certgen:main`
   - **Purpose:** Generates and manages SSL certificates.
   - **Environment Variables:**
     - `KEYSTORE_PW`: Password for the keystore.
     - `TRUSTSTORE_PW`: Password for the truststore.
   - **Volumes:**
     - `key_trust-store-volume`: Stores keystore and truststore files.
     - `keys-vol`: Stores private keys.
     - `certs-vol`: Stores certificate files.
   - **Networks:** Connected to the default network.

### 2. **database**
   - **Image:** `postgres:14.4`
   - **Purpose:** Hosts the PostgreSQL database for OpenELIS Global.
   - **Ports:** Exposes port `15432` for PostgreSQL.
   - **Environment File:** Uses environment variables from `../../volume/database/database.env`.
   - **Volumes:**
     - `db-data`: Persists PostgreSQL data between container restarts.
     - `dbInit`: Runs initialization scripts on container start.
   - **Healthcheck:** Ensures the database is ready before starting dependent services.
   - **Networks:** Connected to the default network.

### 3. **oe.openelis.org**
   - **Image:** `itechuw/openelis-global-2:2.8`
   - **Purpose:** Hosts the OpenELIS Global web application.
   - **Ports:** Exposes ports `8080` for HTTP and `8443` for HTTPS.
   - **Environment Variables:**
     - `DEFAULT_PW`: Default admin password.
     - `TZ`: Time zone setting.
     - `CATALINA_OPTS`: JVM options, including database connection details.
   - **Volumes:**
     - `key_trust-store-volume`: Stores keystore and truststore files.
     - `plugins`: Directory for OpenELIS plugins.
     - `oe_server.xml`: Tomcat server configuration.
     - `OpenELIS-Global.war`: The compiled OpenELIS WAR file.
   - **Secrets:** 
     - `datasource.password`: Database password.
     - `common.properties`: Common configuration properties.
   - **Networks:** Connected to the default network with a specific IPv4 address.

### 4. **fhir.openelis.org**
   - **Image:** `hapiproject/hapi:v6.6.0-tomcat`
   - **Purpose:** Hosts the external FHIR API for OpenELIS Global.
   - **Ports:** Exposes ports `8081` for HTTP and `8444` for HTTPS.
   - **Environment Variables:**
     - `SPRING_CONFIG_LOCATION`: Location of the Spring configuration.
     - `JAVA_OPTS`: JVM options, including SSL configurations.
   - **Volumes:**
     - `key_trust-store-volume`: Stores keystore and truststore files.
     - `hapi_server.xml`: Tomcat server configuration.
   - **Secrets:** 
     - `hapi_application.yaml`: Spring configuration for HAPI FHIR.
   - **Networks:** Connected to the default network.

### 5. **frontend.openelis.org**
   - **Image:** `itechuw/openelis-global-2-frontend-dev:2.8`
   - **Purpose:** Hosts the frontend of the OpenELIS Global application.
   - **Volumes:**
     - `src`: Maps the local source code directory for live development.
     - `public`: Maps the local public directory.
   - **Environment Variables:**
     - `CHOKIDAR_USEPOLLING`: Enables polling for file changes in development.
   - **Networks:** Connected to the default network.

### 6. **proxy**
   - **Image:** `nginx:1.15-alpine`
   - **Purpose:** Acts as a reverse proxy, managing traffic to the OpenELIS Global web application and FHIR API.
   - **Ports:** Exposes ports `80` for HTTP and `443` for HTTPS.
   - **Volumes:**
     - `certs-vol`: Stores SSL certificates for NGINX.
     - `keys-vol`: Stores SSL keys for NGINX.
     - `nginx.conf`: NGINX configuration file.
   - **Networks:** Connected to the default network.

## Network Configuration

The default Docker network is configured with the following settings:

- **Driver:** `bridge`
- **Subnet:** `172.21.1.0/26`

## Volume Definitions

- **db-data:** Persists PostgreSQL data.
- **key_trust-store-volume:** Stores keystore and truststore files for secure communication.
- **certs-vol:** Stores SSL certificates.
- **keys-vol:** Stores SSL keys.

## Secrets Management

Secrets are used to manage sensitive information securely. The following secrets are defined:

- **datasource.password:** Password for the PostgreSQL database.
- **common.properties:** Common properties for the application.
- **hapi_application.yaml:** Configuration for the HAPI FHIR API.

## Deployment

### Steps to Deploy

1. **Clone the repository:**
   ```bash
   git clone <repository-url>
   cd /KAPSIKI-OPENELIS-GLOBAL/deployment/development
   ```

2. **Start the services:**
   ```bash
   docker-compose -f dev.docker-compose.yml up -d
   ```

3. **Check the status of services:**
   ```bash
   docker-compose ps
   ```

4. **Access the applications:**
   - OpenELIS Global Web App: `http://localhost`
   - FHIR API: `http://localhost:8081`

### Stopping the Services

To stop all running containers:

```bash
docker-compose down
```

