
---

# OpenELIS Global Deployment with Docker Compose(dev)

This repository contains a Docker Compose setup for deploying the OpenELIS Global application along with its supporting services. The services include a PostgreSQL database, a web application server, an Nginx proxy, and an external FHIR API server. This setup is intended for environments running on Docker Compose version `3.3`.

## Table of Contents

- [Services](#services)
  - [Certs](#certs)
  - [Database](#database)
  - [Web Application (OpenELIS Global)](#oeopenelisorg)
  - [FHIR API](#fhiropenelisorg)
  - [Frontend](#frontendopenelisorg)
  - [Proxy](#proxy)
- [Secrets](#secrets)
- [Networks](#networks)
- [Volumes](#volumes)
- [Getting Started](#getting-started)
- [Contributing](#contributing)
- [License](#license)

## Services

### Certs
- **Container Name:** `oe-certs`
- **Image:** `itechuw/certgen:main`
- **Environment Variables:**
  - `KEYSTORE_PW`: `"kspass"`
  - `TRUSTSTORE_PW`: `"tspass"`
- **Volumes:**
  - `key_trust-store-volume:/etc/openelis-global`
  - `keys-vol:/etc/ssl/private/`
  - `certs-vol:/etc/ssl/certs/`
- **Networks:** `default`

### Database
- **Container Name:** `openelisglobal-database`
- **Image:** `postgres:14.4`
- **Ports:** `15432:5432`
- **Environment File:** `../../volume/database/database.env`
- **Volumes:**
  - `db-data:/var/lib/postgresql/data`
  - `../../volume/database/dbInit:/docker-entrypoint-initdb.d`
- **Networks:** `default`
- **Healthcheck:**
  - `test: ["CMD", "pg_isready", "-q", "-d", "clinlims", "-U", "clinlims"]`
  - `timeout: 45s`
  - `interval: 10s`
  - `retries: 10`

### Web Application (oe.openelis.org)
- **Container Name:** `openelisglobal-webapp`
- **Image:** `itechuw/openelis-global-2:2.8`
- **Depends On:** `database`, `certs`
- **Ports:**
  - `8080:8080`
  - `8443:8443`
- **Environment Variables:**
  - `DEFAULT_PW=adminADMIN!`
  - `TZ=Africa/Nairobi`
  - `CATALINA_OPTS= -Ddatasource.url=jdbc:postgresql://database:5432/clinlims -Ddatasource.username=clinlims`
- **Volumes:**
  - `key_trust-store-volume:/etc/openelis-global`
  - `../../volume/plugins/:/var/lib/openelis-global/plugins`
  - `../../volume/tomcat/oe_server.xml:/usr/local/tomcat/conf/server.xml`
- **Secrets:** 
  - `datasource.password`
  - `common.properties`
- **Networks:** `default`

### FHIR API (fhir.openelis.org)
- **Container Name:** `external-fhir-api`
- **Build Context:** `../../fhir`
- **Dockerfile:** `./Dockerfile`
- **Depends On:** `database`, `certs`
- **Ports:**
  - `8081:8080`
  - `8444:8443`
- **Environment Variables:**
  - `SPRING_CONFIG_LOCATION: file:///run/secrets/hapi_application.yaml`
  - `TZ: Africa/Nairobi`
  - `JAVA_OPTS: "-Djavax.net.ssl.trustStore=/etc/openelis-global/truststore 
                      -Djavax.net.ssl.trustStorePassword=tspass
                      -Djavax.net.ssl.trustStoreType=pkcs12 
                      -Djavax.net.ssl.keyStore=/etc/openelis-global/keystore 
                      -Djavax.net.ssl.keyStorePassword=kspass 
                      -Djavax.net.ssl.keyStoreType=pkcs12"`
- **Volumes:**
  - `key_trust-store-volume:/etc/openelis-global`
  - `../../volume/tomcat/hapi_server.xml:/opt/bitnami/tomcat/conf/server.xml`
- **Secrets:** 
  - `hapi_application.yaml`
- **Networks:** `default`

### Frontend (frontend.openelis.org)
- **Build Context:** `../../frontend`
- **Dockerfile:** `./Dockerfile`
- **Container Name:** `openelisglobal-front-end`
- **Environment Variables:**
  - `CHOKIDAR_USEPOLLING=true`
- **Networks:** `default`
- **TTY:** `true`

### Proxy
- **Container Name:** `openelisglobal-proxy`
- **Image:** `nginx:1.15-alpine`
- **Ports:**
  - `80:80`
  - `443:443`
- **Volumes:**
  - `certs-vol:/etc/nginx/certs/`
  - `keys-vol:/etc/nginx/keys/`
  - `../../volume/nginx/nginx-prod.conf:/etc/nginx/nginx.conf:ro`
- **Depends On:** `certs`
- **Networks:** `default`
- **Restart Policy:** `unless-stopped`

## Secrets
- **datasource.password:** `../../volume/properties/datasource.password`
- **common.properties:** `../../volume/properties/common.properties`
- **hapi_application.yaml:** `../../volume/properties/hapi_application.yaml`

## Networks
- **default:**
  - **Driver:** `bridge`
  - **IPAM Config:** 
    - `Subnet: 172.20.1.0/24`

## Volumes
- **db-data**
- **key_trust-store-volume**
- **certs-vol**
- **keys-vol**

## Getting Started

1. Clone this repository to your local machine.
2. Navigate to the directory containing the `docker-compose.yml` file fot the dev ,  that is  `deployment/dev`.
3. Make sure all required secrets are placed in the `../../volume/properties/` directory.
4. Run `docker-compose up -d` to start all services.
5. Access the application via your browser at `http://localhost` or `https://localhost`.

## Contributing

Contributions are welcome! Please fork the repository and create a pull request for any changes.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

--- 

This README provides a detailed overview of the Docker Compose setup, including instructions for starting the services.