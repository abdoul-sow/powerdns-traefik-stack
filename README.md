# PowerDNS Docker Stack

This repository contains a Docker-based setup for running PowerDNS with a web-based administration interface (PowerDNS Admin) and MariaDB as the backend database.

## Overview

This stack includes:

- **PowerDNS**: Authoritative DNS server
- **PowerDNS Admin**: Web-based administration interface
- **MariaDB**: Database backend for both PowerDNS and PowerDNS Admin

## Prerequisites

- Docker and Docker Compose installed
- Docker Swarm initialized (for stack deployment)
- Traefik configured (for SSL support)

## Configuration

The environment variables can be set in a `.env` file in the project root. Below are the essential variables:

```
VOLUME_PATH=/opt/powerdns                   # Base path for persistent storage
DOMAIN=dns.example.com                      # Your domain name for PowerDNS Admin

# MariaDB Settings
MARIADB_ROOT_PASSWORD=your_secure_password
MARIADB_USER=powerdns
MARIADB_PASSWORD=your_secure_password
MARIADB_DATABASE=powerdns

# PowerDNS Settings
PDNS_primary=yes
PDNS_webserver=yes
PDNS_webserver_address=0.0.0.0
PDNS_webserver_allow_from=0.0.0.0/0
PDNS_api=yes
PDNS_api_key=your_secure_api_key
PDNS_dnsupdate=true
PDNS_version_string=anonymous

# PowerDNS Admin Settings
GUNICORN_LOGLEVEL=DEBUG
GUNICORN_TIMEOUT=60
GUNICORN_WORKERS=2
SECRET_KEY=your_secure_secret_key
SQLALCHEMY_DATABASE_URI=mysql://powerdns_admin:your_secure_password@mariadb/powerdns_admin
TZ=Africa/Abidjan
```

## Directory Structure

```
powerdns/
├── config
│   └── init.sql       # Initial database setup script
├── docker-compose.yml
├── docker-compose.withssl.yml
├── .env               # Environment variables
├── mariadb-data       # Database persistent storage
└── traefik
    └── letsencrypt
        └── acme.json  # SSL certificate storage
```

## Installation

1. Clone this repository:
   ```
   git clone https://github.com/abdoul-sow/powerdns-traefik-stack.git
   cd powerdns
   ```

2. Create and configure the `.env` file with your settings.

3. Deploy with Docker Swarm:
   ```
   set -a; . .env; set +a && docker stack deploy -c docker-compose.yml -c docker-compose.withssl.yml powerdns
   ```

   This command:
   - Loads the environment variables from `.env`
   - Deploys the stack with both the base configuration and SSL support

## Accessing the Services

- **PowerDNS Admin**: https://your-domain (as specified in the DOMAIN env var)
- **DNS Service**: Available on port 53/udp

## SSL Support

SSL support is provided through Traefik. The `docker-compose.withssl.yml` file contains the Traefik integration configuration.

## Security Considerations

- Change all default passwords in the `.env` file
- Consider restricting access to the PowerDNS Admin interface
- Review the `PDNS_webserver_allow_from` setting to limit access to the API

## Troubleshooting

If you encounter issues:

1. Check the logs:
   ```
   docker service logs powerdns_powerdns
   docker service logs powerdns_powerdns-admin
   docker service logs powerdns_mariadb
   ```

2. Verify that MariaDB is properly initialized:
   ```
   docker exec -it $(docker ps -q -f name=powerdns_mariadb) mysql -u root -p
   ```

3. Ensure all required environment variables are set in your `.env` file

## Maintenance

### Backup

To backup the database:
```
docker exec $(docker ps -q -f name=powerdns_mariadb) mysqldump -u root -p${MARIADB_ROOT_PASSWORD} --all-databases > backup.sql
```

### Updates

To update the stack:
```
docker pull pschiffe/pdns-mysql:alpine
docker pull powerdnsadmin/pda-legacy:latest
docker pull mariadb:lts
set -a; . .env; set +a && docker stack deploy -c docker-compose.yml -c docker-compose.withssl.yml powerdns
```