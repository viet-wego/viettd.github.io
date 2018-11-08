Title: Setup replication for PostgreSQL
Date: 2018-11-07 20:00
Modified: 2018-11-07 20:00
Category: database
Tags: postgresql, ubuntu, linux, devops, replication, stanby, high availability, load balancing
Slug: setup-replication-for-postgresql
Authors: Viet Tran
Summary: How to setup replication for PostgreSQL using hot stanby.

If you want to understand about high availability,load balancing and replication in PostgreSQL, please read the official documentation in the [References](#References). In this tutorial, I will show you how to set up a replication server for PostgreSQL using Hot Stanby mode.

## Install PostgreSQL

You will need 2 instances of PostgreSQL. For detail please see [Install & configure PostgreSQL on Ubuntu](https://viettran.me/install-n-configure-postgresql-on-ubuntu.html).

## Prepare Master Server

1. **Create a user for replication & allow traffic for replication server**

        sudo -u postgres createuser -U postgres rep_user -P -c 5 --replication
    Replace `rep_user` with your replication user.
    Replace `5` with max connections can be used for replication.
    Your will need to provide a powerful password for this user at the command prompt.

    - Edit **pg_hba.conf** file. By default, you can find it at `/etc/postgresql/[your_postgresql_version]/main/pg_hba.conf`.

                sudo vi /etc/postgresql/10/main/pg_hba.conf

    - Allow replication connections

                # Allow replication connections for user rep_user from 10.240.0.0/16 with md5 password
                host    replication     rep_user             10.240.0.0/16           md5
        If you want to only allow from 1 specific IP, can use CIDR `[your_replication_server_ip]/32`. E.g: 10.240.0.10/32

2. **Configure WAL (Write Ahead Log)**
    - Create a directory to store archive file

            mkdir -p /data/postgresql/10/main/archive

    - Edit **postgresql.conf**

            sudo vi /etc/postgresql/10/main/postgresql.conf
        - PostgreSQL >= 9.6

                wal_level = replica

        - PostgreSQL <= 9.5

                wal_level = hot_standby

        - Enable archive mode

                archive_mode = on
                archive_command = 'test ! -f /data/postgresql/10/main/archive/%f && cp %p /data/postgresql/10/main/archive/%f'

            *Replace **/data/postgresql/10/main/archive** with the directory you created above.*

    - Restart service to make the changes effective

            sudo service postgresql restart

## Prepare Replication Server

1. **Stop service**

        sudo service postgresql stop

2. **Initialize data**

    - Rename old data directory

                # replace /data/postgresql/10/main with your postgresql data directory (configured in postgresql.conf)
                sudo mv /data/postgresql/10/main /data/postgresql/10/main_old

    - Run backup utility to copy data from Master

                sudo -u postgres pg_basebackup -h  -D /data/postgresql/10/main -U rep_user -v -P -X stream

        *Replace **/data/postgresql/10/main** with your data directory & **rep_user** with your replication user above. You will need to provide password for replication user.*

    - Configure replication

        - Turn on Hot Standby

                sudo vi  /etc/postgresql/10/main/postgresql.conf

            Uncomment this line

                hot_standby = on

        - Create recovery file

                sudo cp /usr/share/postgresql/10/recovery.conf.sample /data/postgresql/10/main/recovery.conf
                sudo vi /data/postgresql/10/main/recovery.conf

            Configure stanby

                standby_mode = on
                primary_conninfo = 'host=[your_master_host] port=[your_master_port] user=[your_replication_user] password=[your_replication_user_password]'

            Make sure user **postgres** has correct permission on postgresql data directory

                sudo chmod -R 0700 /data/postgresql
                sudo chown -R postgres:postgres /data/postgresql

3. **Start service**

        sudo service postgresql start

Now your **Replication Server** can serve read-only query. If you want to promote it to **Master** can use this command:

        sudo -u postgres pg_ctlcluster 10 main promote

*Replace `10` with your postgresql version*.

---

## References

- [How to Set Up PostgreSQL for High Availability and Replication with Hot Standby](https://cloud.google.com/community/tutorials/setting-up-postgres-hot-standby)

- [High Availability, Load Balancing, and Replication (PostgreSQL's official documentation)](https://www.postgresql.org/docs/10/high-availability.html)