# Node.js Setup Guide

This guide walks through the steps to set up Node.js using NVM:
1. Installs curl for downloading resources
2. Installs NVM (Node Version Manager)
3. Installs the latest LTS version of Node.js
4. Installs global dependencies like PM2 and TypeScript
5. Verifies all installations are working correctly

## Prepare installation of postgresql (17)
```bash
## Add postgresql repository
sudo sh -c 'echo "deb http://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/pgdg.list'
## Import the repository signing key
curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg
```

## Install postgresql (17)
```bash
sudo apt update
sudo apt install postgresql-17
```

## Verify postgresql installation
```bash
sudo systemctl status postgresql
psql --version
```

## Check postgresql status
```bash
## Start and enable PostgreSQL service:
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

## Install Node.js LTS version
```bash
sudo cp /etc/postgresql/17/main/postgresql.conf /etc/postgresql/17/main/postgresql.conf.backup
sudo nano "/etc/postgresql/17/main/postgresql.conf"

ls -l /etc/postgresql/17/main/postgresql.conf
sudo chown be4c0n /etc/postgresql/17/main/postgresql.conf
sudo chown postgres:postgres /etc/postgresql/17/main/postgresql.conf
```
## PostgreSQL Configuration Parameters
[
  {
    "name": "listen_addresses",
    "value": "*",
    "description": "Allow connections from other machines (restrict via pg_hba.conf)"
  },
  {
    "name": "max_connections",
    "value": "200",
    "description": "Reasonable starting point, can be increased if needed"
  },
  {
    "name": "shared_buffers",
    "value": "32GB",
    "description": "~25% of total RAM"
  },
  {
    "name": "effective_cache_size",
    "value": "96GB",
    "description": "~75% of total RAM"
  },
  {
    "name": "maintenance_work_mem",
    "value": "2GB",
    "description": "For maintenance operations (VACUUM, CREATE INDEX)"
  },
  {
    "name": "work_mem",
    "value": "128MB",
    "description": "Per sort/hash operation (watch total: work_mem * max_connections)"
  },
  {
    "name": "wal_buffers",
    "value": "16MB",
    "description": "Or -1 for auto-adjustment based on shared_buffers"
  },
  {
    "name": "min_wal_size",
    "value": "4GB",
    "description": "Minimum WAL size"
  },
  {
    "name": "max_wal_size",
    "value": "16GB",
    "description": "Maximum WAL size"
  },
  {
    "name": "default_statistics_target",
    "value": "100",
    "description": "Can be increased to 500-1000 if query plans aren't optimal"
  },
  {
    "name": "random_page_cost",
    "value": "1.0",
    "description": "For SSD NVMe storage"
  },
  {
    "name": "effective_io_concurrency",
    "value": "200",
    "description": "For SSD storage"
  },
  {
    "name": "max_worker_processes",
    "value": "8",
    "description": "Number of CPU cores"
  },
  {
    "name": "max_parallel_workers_per_gather",
    "value": "4",
    "description": "Parallel workers per query (half of cores)"
  },
  {
    "name": "max_parallel_maintenance_workers",
    "value": "4",
    "description": "For parallel maintenance"
  },
  {
    "name": "max_parallel_workers",
    "value": "8",
    "description": "Total parallel workers"
  }
]

##
```bash
sudo cp "/etc/postgresql/17/main/pg_hba.conf" "/etc/postgresql/17/main/pg_hba.conf.backup"
sudo nano "/etc/postgresql/17/main/pg_hba.conf"

ls -l /etc/postgresql/17/main/pg_hba.conf
sudo chown be4c0n /etc/postgresql/17/main/pg_hba.conf
sudo chown postgres:postgres /etc/postgresql/17/main/pg_hba.conf
sudo chmod u+w /etc/postgresql/17/main/pg_hba.conf
sudo chmod 644 /etc/postgresql/17/main/pg_hba.conf
```

## pg_hba.conf
```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# Login by Unix domain socket (peer authentication)
local   all             postgres                                scram-sha-256
local   all             admin                                   scram-sha-256
local   all             all                                     scram-sha-256

# Superutilisateur 'admin' - accès via loopback TCP/IP (pour tunnels SSH depuis la machine locale)
host    all             admin           127.0.0.1/32            scram-sha-256

# Utilisateur global 'app_rw' - accès distant avec mot de passe
host    all             +readwrite_users_group    0.0.0.0/0     scram-sha-256
host    all             +readwrite_users_group    ::/0          scram-sha-256

# Utilisateurs en lecture seule (via un rôle, ex: 'readonly_users_group') - accès distant avec mot de passe
# Nous appliquerons les restrictions de connexion au niveau du rôle plus tard
host    all             +readonly_users_group    0.0.0.0/0      scram-sha-256
host    all             +readonly_users_group    ::/0           scram-sha-256

# Refuser toute autre tentative de connexion pour l'utilisateur 'admin' depuis le réseau
# (Optionnel, mais ajoute une couche de sécurité si listen_addresses = '*' est large)
host    all             admin           0.0.0.0/0               reject
host    all             admin           ::/0                    reject
```

## Verify installations
```bash
sudo -u postgres psql
```

```bash
ALTER USER postgres PASSWORD 'unMotDePasseTresFort';

CREATE ROLE admin WITH LOGIN SUPERUSER PASSWORD 'unMotDePasseTresFort';

CREATE ROLE readwrite_users_group;
CREATE ROLE yogaa WITH LOGIN PASSWORD 'unMotDePasseTresFort' IN ROLE readwrite_users_group;

CREATE ROLE readonly_users_group;
CREATE ROLE slum WITH LOGIN PASSWORD 'unMotDePasseTresFort' IN ROLE readonly_users_group CONNECTION LIMIT 2;

# 1. Pour readwrite_users_group (donc yogaa) :
ALTER DEFAULT PRIVILEGES FOR ROLE admin
  IN SCHEMA public
  GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO readwrite_users_group;

ALTER DEFAULT PRIVILEGES FOR ROLE admin
  IN SCHEMA public
  GRANT USAGE, SELECT, UPDATE ON SEQUENCES TO readwrite_users_group;

ALTER DEFAULT PRIVILEGES FOR ROLE admin
  IN SCHEMA public
  GRANT EXECUTE ON FUNCTIONS TO readwrite_users_group;

# 2. Pour readonly_users_group (donc slum) :
ALTER DEFAULT PRIVILEGES FOR ROLE admin
    IN SCHEMA public
    GRANT SELECT ON TABLES TO readonly_users_group;

ALTER DEFAULT PRIVILEGES FOR ROLE admin
    IN SCHEMA public
    GRANT SELECT ON SEQUENCES TO readonly_users_group;

ALTER DEFAULT PRIVILEGES FOR ROLE admin
    IN SCHEMA public
    GRANT EXECUTE ON FUNCTIONS TO readonly_users_group;

\q

```bash


# Create a new database
sudo systemctl restart postgresql
sudo ufw allow 5432/tcp comment 'PostgreSQL'
sudo ufw allow from VOTRE_IP_AUTORISEE to any port 5432 proto tcp
sudo ufw reload

##
sudo rm /etc/fail2ban/filter.d/postgresql.conf
sudo nano /etc/fail2ban/filter.d/postgresql.conf

[Definition]
datepattern = ^\s*%%Y-%%m-%%d(?:T|  ?)%%H:%%M:%%S(?:\.%%f)?(?:\s*%%Z)?
__prefix_line = \s*\[\d+\] postgres@\S+\s+
failregex = ^%(__prefix_line)sFATAL:  password authentication failed for user "<ADDR>"
            ^%(__prefix_line)sFATAL:  no pg_hba.conf entry for host "<ADDR>", user "<F-USER>[^"]+</F-USER>"


##
sudo rm /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

[postgresql-auth]
enabled  = true
port     = 5432
filter   = postgresql
logpath  = /var/log/postgresql/postgresql-*.log
maxretry    = 3
findtime    = 60
bantime     = 60

sudo systemctl restart fail2ban

### Check erroR
sudo tail -n 50 /var/log/fail2ban.log


sudo systemctl status fail2ban
sudo fail2ban-client status
sudo fail2ban-client status postgresql-auth