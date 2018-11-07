Title: Install & configure PostgreSQL on Ubuntu
Date: 2018-10-08 13:40
Modified: 2018-10-08 18:59
Category: database
Tags: postgresql, ubuntu, linux, devops
Slug: install-n-configure-postgresql-on-ubuntu
Authors: Viet Tran
Summary: How to install & configure PostgreSQL on Ubuntu.

*This tutorial is using [vi](https://en.wikipedia.org/wiki/Vi) editor. You can use any editor you want.*

### Add PostgreSQL repository to **apt** package list

Ubuntu comes with a default version of PostgreSQL. If we need to install a specific version, we have to add the PosgreSQL repository to our package list.

1. Edit this file:

        sudo vi /etc/apt/sources.list.d/pgdg.list

1. Add this line (replace `xenial` with your Ubuntu code name, I use 16.04):

        deb http://apt.postgresql.org/pub/repos/apt/ xenial-pgdg main

1. Import the repository signing key, and update the package list:

        wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
        sudo apt-get update

### Install PostgreSQL

1. Our current system requires PostgreSQL 9.5. So I will install version **9.5**. Replace it with your choice:

        sudo apt-get install -y postgresql-client-9.5 postgresql-9.5 postgresql-contrib-9.5 postgresql-server-dev-9.5

    *From version **10** you don't have to install **postgresql-contrib-xx** because it is already added to package **postgresql-xx***

1. PostgreSQL created a default user, named **postgres**, during installation. This user doesn't yet have a password, so you'll need to set one.

    - Run **PSQL** as user **postgres**, instead of **root**, accesssing the database named **postgres**:

            sudo -u postgres psql postgres

    - You should see the PSQL command prompt, which looks like this: `postgres=#`.
    Set the password with this command:

            \password postgres

        When prompted, enter and confirm the password you've chosen

    - Install the **adminpack** extension to use with some administration and management tools. The console prints **CREATE EXTENSION** when successful.

            CREATE EXTENSION adminpack;

    - Enter `\q` to exit **PSQL**

### Allow access to PostgreSQL

PostgreSQL only allows access from localhost. We need to change its configuratin to allow remote access.

1. Edit **pg_hba.conf**

        sudo vi /etc/postgresql/9.4/main/pg_hba.conf
    This file contains a very clear information. You can base on this to add your configuration. In this tutorial, I will allow access from all clients by adding this:

        # allow access from my internal network with md5 password
        host    all             10.20.0.0/16             all                     md5


2. Edit **postgresql.conf**

        sudo vi /etc/postgresql/9.4/main/postgresql.conf
    Find this `#listen_addresses = 'localhost'` & replace with `listen_addresses = '*'` to allow access from other servers.

3. Save the file & exit. Then restart **postgresql** service:

        sudo service postgresql restart

### Configure PostgreSQL

Below is basic settings for **postgresql.conf**. Please see [References](#References) for more detail.

1. Memory:

        shared_buffers = 3GB         # 1/4 system available RAM
        work_mem = 10MB                 # up to 1/4(RAM)/max_connections

1. Query turning

        random_page_cost = 1.5          # using 1.5 or 2.0 for SSD
        effective_cache_size = 10GB     # 3/4 system available RAM or sum of free & cached value from free -h command

1. Remember to save the file and restart postgresql service.

---

### References

- [How to Set Up PostgreSQL on Google Compute Engine](https://cloud.google.com/community/tutorials/setting-up-postgres)
- [Setting Up an Optimal Environment for PostgreSQL](https://severalnines.com/blog/setting-optimal-environment-postgresql)