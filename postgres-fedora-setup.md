# Installing PostgreSQL 17 at Fedora 43
by: [@kauanpecanha](https://github.com/kauanpecanha)

Description: These instructions are intended to teach the process to install postgresql at Fedora 43.

## Specs
OS: fedora 43

## Instructions
Warning: these instructions might be outdated. be aware with the specific version used.

Download the needed release with the commands:
```bash
sudo dnf install -y https://download.postgresql.org/pub/repos/yum/reporpms/F-43-x86_64/pgdg-fedora-repo-latest.noarch.rpm
sudo dnf install -y postgresql17-server
sudo /usr/pgsql-17/bin/postgresql-17-setup initdb
sudo systemctl enable postgresql-17
sudo systemctl start postgresql-17
```

To prevent the error bellow, you should do the following steps:

```bash
psql: erro: a conexão com o servidor no soquete "/run/postgresql/.s.PGSQL.5432" falhou: FATAL: A autenticação do tipo peer falhou para o usuário "postgres"
```

Edit the config file
```bash
sudo nano /var/lib/pgsql/17/data/pg_hba.conf
```

And change all the peer mentions by scram-sha-256, just like bellow:
```bash
# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     scram-sha-256
# IPv4 local connections:
host    all             all             127.0.0.1/32            scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     scram-sha-256
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```

Get postgres service name
```bash
systemctl list-units | grep -i postgres
```

Reload it
```bash
sudo systemctl status postgresql-17.service
```

Now you are able of doing it, using the command bellow:
```bash
psql -U postgres -W
```

## Reference
- [postgres official documentation red hat family downloads page](https://www.postgresql.org/download/linux/redhat/)

Made with ❤️ in 🇧🇷