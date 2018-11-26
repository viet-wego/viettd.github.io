Title: Install & configure PostgreSQL on Ubuntu
Date: 2018-10-08 13:40
Modified: 2018-11-25 14:29
Category: database
Tags: postgresql, ubuntu, linux, devops
Slug: install-n-configure-postgresql-on-ubuntu
Authors: Viet Tran
Summary: How to install & configure PostgreSQL on Ubuntu.

*This tutorial is using [vi](https://en.wikipedia.org/wiki/Vi) editor. You can use any editor you want.*

## Add PostgreSQL repository to **apt** package list

Ubuntu comes with a default version of PostgreSQL. If we need to install a specific version, we have to add the PosgreSQL repository to our package list.

```bash
sudo vi /etc/apt/sources.list.d/pgdg.list
```

Add this line:

```bash
deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main
```

*Replace `xenial` with your Ubuntu code name, I use `xenial` because I'm using Ubuntu 16.04 LTS*

Import the repository signing key, and update the package list:

```bash
wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
sudo apt-get update
```

## Install PostgreSQL

Our current system requires PostgreSQL 9.5. So I will install version **9.5**. Replace it with your choice:

```bash
sudo apt-get install -y postgresql-client-9.5 postgresql-9.5 postgresql-contrib-9.5 postgresql-server-dev-9.5
```

*From version **10** you don't have to install **postgresql-contrib-xx** because it is already added to package **postgresql-xx***

PostgreSQL created a default user, named **postgres**, during installation. This user doesn't yet have a password, so you'll need to set one.
Run **PSQL** as user **postgres**, instead of **root**, accesssing the database named **postgres**:

```bash
sudo -u postgres psql postgres
```

You should see the PSQL command prompt, which looks like this: `postgres=#`. Set the password with this command:

```bash
\password postgres
```

When prompted, enter and confirm the password you've chosen.

Install the **adminpack** extension to use with some administration and management tools.

```bash
CREATE EXTENSION adminpack;
```

The console prints **CREATE EXTENSION** when successful.

Enter `\q` to exit **PSQL**

## Allow access to PostgreSQL

PostgreSQL only allows access from localhost. We need to change its configuratin to allow remote access.

##### Edit **pg_hba.conf**

```bash
sudo vi /etc/postgresql/9.4/main/pg_hba.conf
```

This file contains a very clear information. You can base on this to add your configuration. In this tutorial, I will allow access from all users in my internal network by adding this:

```bash
# allow access from my internal network with md5 password
host    all             10.20.0.0/16             all                     md5
```

##### Edit **postgresql.conf**

```bash
sudo vi /etc/postgresql/9.4/main/postgresql.conf
```

Find this `#listen_addresses = 'localhost'` & replace with `listen_addresses = '*'` to allow access from other servers.

Save the file & exit. Then restart **postgresql** service:

```bash
sudo service postgresql restart
```

## Configure PostgreSQL

Below is basic settings for **postgresql.conf**. Please see [References](#References) for more detail.

##### Memory

```bash
shared_buffers = 3GB         # 1/4 system available RAM
work_mem = 10MB                 # up to 1/4(RAM)/max_connections
```

##### Query turning

```bash
random_page_cost = 1.5          # using 1.5 or 2.0 for SSD
effective_cache_size = 10GB     # 3/4 system available RAM or sum of free & cached value from free -h command
```

*Remember to save the file and restart postgresql service.*

---

## References

- [How to Set Up PostgreSQL on Google Compute Engine](https://cloud.google.com/community/tutorials/setting-up-postgres)
- [Setting Up an Optimal Environment for PostgreSQL](https://severalnines.com/blog/setting-optimal-environment-postgresql)