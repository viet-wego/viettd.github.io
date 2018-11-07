Title: Setup replication for PostgreSQL
Date: 2018-11-07 20:00
Modified: 2018-11-07 20:00
Category: database
Tags: postgresql, ubuntu, linux, devops, replication, stanby, high availability, load balancing
Slug: setup-replication-for-postgresql
Authors: Viet Tran
Summary: How to setup replication for PostgreSQL using hot stanby.


If you want to understand about high availability,load balancing and replication in PostgreSQL, please read the official documentation in the [References](#References). In this tutorial, I will show you how to set up a replication server for PostgreSQL using Hot Stanby mode.

### Install PostgreSQL

You will need 2 instances of PostgreSQL. For detail please see [Install & configure PostgreSQL on Ubuntu](https://viettran.me/install-n-configure-postgresql-on-ubuntu.html).

### Prepare Master Node

1. Create a user for replication & allow traffic for replication server

        sudo -u postgres createuser -U postgres rep_user -P -c 5 --replication
    - Replace `rep_user` with your replication user.
    - Replace `5` with max connections can be used for replication.
    - Your will need to provide a powerful password for this user at the command prompt.
    
    Edit **pg_hba.conf** file. By default, you can find it at `/etc/postgresql/[your_postgresql_version]/main/pg_hba.conf`.

        sudo vi /etc/postgresql/10/main/pg_hba.conf
    - Allow repication connections

            # Allow replication connections for user rep_user from 10.240.0.0/16 with md5 password
            host    replication     rep_user             10.240.0.0/16           md5
        If you want to only allow from 1 specific IP, can use CIDR `[your_replication_server_ip]/32`. Ex: 10.240.0.10/32

2. Configurate WAL
    - Create a directory to store archive file
            
            mkdir -p /data/postgresql/10/main/archive
    - Edit **postgresql.conf**

            sudo vi /etc/postgresql/10/main/postgresql.conf
        - WAL level
                
                # postgresql >= 9.6
                wal_level = replica

---

### References

- [How to Set Up PostgreSQL for High Availability and Replication with Hot Standby](https://cloud.google.com/community/tutorials/setting-up-postgres-hot-standby)

- [High Availability, Load Balancing, and Replication (PostgreSQL's official documentation)](https://www.postgresql.org/docs/10/high-availability.html)