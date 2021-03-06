# WSO2 EI 6.1.1 Bootstrap Script
This script configures different runtimes in WSO2 Enterprise Integrator 6.1.1.

![WSO2 EI Minimum Deployment](img/pattern.png "WSO2 EI Minimum Deployment")

## Supported Runtimes
1. WSO2 EI Integrator
2. WSO2 EI BPS

## How To
1. Download and extract the WSO2 EI 6.1.1 distribution. This location would be the value to be used in the configuration option `WSO2_CARBON_HOME`.
2. Download JDK 1.8 and setup `JAVA_HOME`. Optionally add `$JAVA_HOME/bin` to `$PATH`.
3. Clone this repository to a preferred location.
```bash
git clone https://github.com/chamilad/wso2ei-conf.git
```
4. Download any JDBC drivers and other external libraries that are needed to be copied over to `CARBON_HOME` and place them inside the `files` folder as described in the section [Copying Files](#copying-files).
  * [MySQL JDBC Driver](https://dev.mysql.com/downloads/connector/j/)
  * [Postgres JDBC Driver](https://jdbc.postgresql.org/download.html)

  ```bash
  # Example: Download and copy the PostgreSQL JDBC Driver to all the runtimes when configuring.

  # 1. Create the intended folder structure relative to CARBON_HOME
  cd wso2ei-conf
  mkdir -p files/common/lib

  # 2. Download the JDBC Driver
  wget https://jdbc.postgresql.org/download/postgresql-42.1.4.jar -P files/common/lib
  ```
5. Setup databases as described in the section [Database Configuration](#database-configuration).
6. **OPTIONAL:** Setup any load balancers that would front these instances.
> If any load balancers are created, configurations `WSO2_HOSTNAME`, `WSO2_HTTP_PROXY_PORT`, and `WSO2_HTTPS_PROXY_PORT` will have to be set

7. Set necessary environment variables as described in the following section (scripting the configuration options is described in [another section](#running-the-setupsh)).
8. Run `setup.sh` file.

## Detailed How-to
### Configuration Options
The configuration options are extracted from environment variables.

1. `WSO2_SERVER_RUNTIME` - The runtime id of the server
  * WSO2 EI BPS - `business-process`
  * WSO2 EI Integrator - `integrator`
2. `WSO2_CARBON_HOME` - The location where the WSO2 Server is located
3. `WSO2_HOSTNAME` - The hostname to be used in the WSO2 servers
4. `WSO2_REG_DB_HOSTNAME` - The hostname of the registry DB server
5. `WSO2_REG_DB_PORT` - The port at which the registry DB server is operating
6. `WSO2_REG_DB_NAME` - The registry database name
7. `WSO2_REG_DB_PROTOCOL` - The JDBC URL Protocol to use for the registry database. This would use a default value of `mysql`
  * MySQL - `mysql`
  * Postgres - `postgresql`
8. `WSO2_REG_DB_DRIVER_NAME` - The JDBC driver name to be used in the datasources for the registry database. This would use a default value of `com.mysql.jdbc.Driver`
  * MySQL - `com.mysql.jdbc.Driver`
  * Postgres - `org.postgresql.Driver`
9. `WSO2_REG_DB_USERNAME` - The username to access the registry DB
10. `WSO2_REG_DB_PASSWORD` - The password to access the registry DB
11. `WSO2_BPS_DB_HOSTNAME` - The hostname of the BPS DB server
12. `WSO2_BPS_DB_PORT` - The port at which the BPS DB server is operating
13. `WSO2_BPS_DB_NAME` - The BPS database name
14. `WSO2_BPS_DB_PROTOCOL` - The JDBC URL Protocol to use for the BPS database. This would use a default value of `mysql`
  * MySQL - `mysql`
  * Postgres - `postgresql`
15. `WSO2_BPS_DB_DRIVER_NAME` - The JDBC driver name to be used in the datasources for the BPS database. This would use a default value of `com.mysql.jdbc.Driver`
  * MySQL - `com.mysql.jdbc.Driver`
  * Postgres - `org.postgresql.Driver`
16. `WSO2_BPS_DB_USERNAME` - The username to access the BPS DB
17. `WSO2_BPS_DB_PASSWORD` - The password to access the BPS DB
18. `WSO2_SERVER_ARGS` - Arguments to pass to the WSO2 Server starter script
19. `WSO2_HTTP_PROXY_PORT` - The `proxyPort` value to be used for the HTTP port. These values, if present, would be used in `catalina-server.xml` file.
20. `WSO2_HTTPS_PROXY_PORT` - The `proxyPort` value to be used for the HTTPS port. These values, if present, would be used in `catalina-server.xml` file.

Following is a sample set of values that configure a WSO2 EI 6.1.1 instance with the following parameters.
1. **Product**: WSO2 EI 6.1.1 BPS
2. **DB**: Postgres (preconfigured), two databases, for BPS and Registry
3. **DB Host**: `localhost`
4. **Load balancer setup**: true

```bash
export WSO2_SERVER_RUNTIME="business-process"

export WSO2_CARBON_HOME="/home/chamilad/dev/wso2ei-6.1.1"
export WSO2_SERVER_ARGS=""

# registry database
export WSO2_REG_DB_HOSTNAME="localhost"
export WSO2_REG_DB_PORT="5432"
export WSO2_REG_DB_NAME="WSO2REG_DB"
export WSO2_REG_DB_USERNAME="postgres"
export WSO2_REG_DB_PASSWORD="toor"
export WSO2_REG_DB_DRIVER_NAME="org.postgresql.Driver"
export WSO2_REG_DB_PROTOCOL="postgresql"

# "business-process" specific configurations
export WSO2_BPS_DB_HOSTNAME="localhost"
export WSO2_BPS_DB_PORT="5432"
export WSO2_BPS_DB_NAME="WSO2BPS_DB"
export WSO2_BPS_DB_USERNAME="postgres"
export WSO2_BPS_DB_PASSWORD="toor"
export WSO2_BPS_DB_DRIVER_NAME="org.postgresql.Driver"
export WSO2_BPS_DB_PROTOCOL="postgresql"

# load balancer related configurations
export WSO2_HOSTNAME="bps-lb.example.com"
export WSO2_HTTP_PROXY_PORT="80"
export WSO2_HTTPS_PROXY_PORT="443"
```

### Database Configuration
This script expects the databases to be already created and populated with the initial source scripts.

#### Setting up the Database
For each runtime, the initial database scripts are shipped with the product. These would be inside `CARBON_HOME/wso2/RUNTIME/dbscripts` folder (except for the Integration runtime in which the database scripts are in `CARBON_HOME/dbscripts`). After creating the database, the relevant `*.sql` files can be sourced in. How these should be restored differ based on the type of the RDBMS.

For example, setting up a Postgre databases with data needed for WSO2 EI BPS would require steps similar to the following.

1. Login to Posgres.
```bash
sudo -u postgres psql postgres
```
2. Create databases and connect.
```bash
psql (9.6.5, server 9.5.8)
Type "help" for help.

postgres=# \l
                                  List of databases
    Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
------------+----------+----------+-------------+-------------+-----------------------
 postgres   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
 template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
            |          |          |             |             | postgres=CTc/postgres
 template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
            |          |          |             |             | postgres=CTc/postgres
(3 rows)

postgres=# CREATE DATABASE "WSO2BPS_DB";
CREATE DATABASE
postgres=# \l
List of databases
Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
------------+----------+----------+-------------+-------------+-----------------------
WSO2BPS_DB | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
postgres   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
|          |          |             |             | postgres=CTc/postgres
template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
|          |          |             |             | postgres=CTc/postgres
(4 rows)

postgres=# \c WSO2BPS_DB;
psql (9.6.5, server 9.5.8)
You are now connected to database "WSO2BPS_DB" as user "postgres".
WSO2BPS_DB=# \dt
No relations found.
WSO2BPS_DB=#
...
...
WSO2BPS_DB=# CREATE DATABASE "WSO2REG_DB";
CREATE DATABASE
WSO2BPS_DB=# \l
List of databases
Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges
------------+----------+----------+-------------+-------------+-----------------------
WSO2BPS_DB | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
WSO2REG_DB | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
postgres   | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 |
template0  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
|          |          |             |             | postgres=CTc/postgres
template1  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
|          |          |             |             | postgres=CTc/postgres
(5 rows)

WSO2BPS_DB=# \c WSO2REG_DB;
psql (9.6.5, server 9.5.8)
You are now connected to database "WSO2REG_DB" as user "postgres".
WSO2REG_DB=# \dt
No relations found.
WSO2REG_DB=#
```
3. Execute the database scripts for BPEL and BPMN on top of `WSO2BPS_DB`.
```bash
postgres=# \c WSO2BPS_DB;
psql (9.6.5, server 9.5.8)
You are now connected to database "WSO2BPS_DB" as user "postgres".
WSO2BPS_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/bps/bpel/create/postgresql.sql
...
...
WSO2BPS_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/bpmn/create/activiti.postgres.create.engine.sql
...
...
WSO2BPS_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/bpmn/create/activiti.postgres.create.history.sql
...
...
WSO2BPS_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/bpmn/create/activiti.postgres.create.identity.sql
...
...
WSO2BPS_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/bpmn/create/activiti.postgres.create.substitute.sql
...
...
WSO2BPS_DB=# \dt
List of relations
Schema |          Name           | Type  |  Owner
--------+-------------------------+-------+----------
public | act_bps_substitutes     | table | postgres
public | act_evt_log             | table | postgres
public | act_ge_bytearray        | table | postgres
public | act_ge_property         | table | postgres
public | act_hi_actinst          | table | postgres
public | act_hi_attachment       | table | postgres
public | act_hi_comment          | table | postgres
public | act_hi_detail           | table | postgres
public | act_hi_identitylink     | table | postgres
public | act_hi_procinst         | table | postgres
public | act_hi_taskinst         | table | postgres
public | act_hi_varinst          | table | postgres
public | act_id_group            | table | postgres
public | act_id_info             | table | postgres
public | act_id_membership       | table | postgres
public | act_id_user             | table | postgres
public | act_procdef_info        | table | postgres
...
...
...
WSO2BPS_DB=#
```
4. Execute the database scripts for WSO2 Carbon on top of `WSO2REG_DB`.
```bash
postgres=# \c WSO2REG_DB;
psql (9.6.5, server 9.5.8)
You are now connected to database "WSO2REG_DB" as user "postgres".
WSO2REG_DB=# \i /home/chamilad/wso2ei-6.1.1/wso2/business-process/dbscripts/postgresql.sql
...
...
WSO2REG_DB=# \dt
List of relations
Schema |         Name          | Type  |  Owner   
--------+-----------------------+-------+----------
public | reg_association       | table | postgres
public | reg_cluster_lock      | table | postgres
public | reg_comment           | table | postgres
public | reg_content           | table | postgres
public | reg_content_history   | table | postgres
public | reg_log               | table | postgres
public | reg_path              | table | postgres
public | reg_property          | table | postgres
public | reg_rating            | table | postgres
public | reg_resource          | table | postgres
public | reg_resource_comment  | table | postgres
public | reg_resource_history  | table | postgres
public | reg_resource_property | table | postgres
public | reg_resource_rating   | table | postgres
public | reg_resource_tag      | table | postgres
public | reg_snapshot          | table | postgres
public | reg_tag               | table | postgres
public | um_account_mapping    | table | postgres
public | um_claim              | table | postgres
public | um_claim_behavior     | table | postgres
public | um_dialect            | table | postgres
public | um_domain             | table | postgres
public | um_hybrid_remember_me | table | postgres
public | um_hybrid_role        | table | postgres
public | um_hybrid_user_role   | table | postgres
public | um_module             | table | postgres
public | um_module_actions     | table | postgres
public | um_permission         | table | postgres
public | um_profile_config     | table | postgres
public | um_role               | table | postgres
public | um_role_permission    | table | postgres
public | um_shared_user_role   | table | postgres
public | um_system_role        | table | postgres
public | um_system_user        | table | postgres
public | um_system_user_role   | table | postgres
public | um_tenant             | table | postgres
public | um_user               | table | postgres
public | um_user_attribute     | table | postgres
public | um_user_permission    | table | postgres
public | um_user_role          | table | postgres
(40 rows)

WSO2REG_DB=#
```

##### Setup Databases Automatically
If the manual setup of the database cannot be done for some reason, the option to setup databases automatically when the product starts can be followed. For this, pass the `-Dsetup` system property to the WSO2 Server starter script.

> Please note that using `-Dsetup` is not recommended for a production environment. Properly setting up databases should be done with database scripts that are shipped with the product

```bash
cd $CARBON_HOME/bin
./business-process.sh -Dsetup
```

If this system property is passed, WSO2 Carbon will execute the relevant database scripts on top of the databases specified. However, this would not always result in expected state, especially if the databases specified are not empty.

`WSO2_SERVER_ARGS` can be used to pass `-Dsetup` argument into the script.

```bash
export WSO2_SERVER_ARGS="-Dsetup"
```

### Copying Files
Create the directory structure inside the `files/<runtime>` folder, as the file should be copied to the destination, relative to `CARBON_HOME`. For example, if the MySQL JDBC driver should be copied inside `$CARBON_HOME/lib` folder for the `business-process` runtime, then the folder structure inside `files` folder should be as follows.

```
files
├── business-process
│   └── lib
|     └──mysql-connector-java-5.1.39-bin.jar
├── common
└── integrator
```

Files copied in the above manner in to the `common` folder will be copied into every runtime. If files exist with the same file name in both the `common` folder and the runtime specific folder, the file in the runtime specific folder would be used.

### Running the `setup.sh`
```bash
# Make a copy of the provided sample configuration file conf.sh.sample as conf.sh.
$ cp conf.sh.sample conf.sh

# Add the properites as needed.
$ vi conf.sh

# Source the conf.sh.
$ source conf.sh

# Run setup.sh file. This would start the WSO2 Server at the above specified location.
$ bash setup.sh
Copying template configurations...
Making common configuration changes...
Making BPS specific configuration changes...
Copying files...
Starting WSO2 Server...
JAVA_HOME environment variable is set to /home/chamilad/dev/java/jdk1.8.0_77
CARBON_HOME environment variable is set to /home/chamilad/dev/QSP/Gartner-2017/wso2ei-6.1.1/wso2/business-process
Using Java memory options: -Xms256m -Xmx1024m
[2017-09-22 10:46:23,854] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Starting WSO2 Carbon...
[2017-09-22 10:46:23,858] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Operating System : Linux 4.4.0-96-generic, amd64
[2017-09-22 10:46:23,858] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Java Home        : /home/chamilad/dev/java/jdk1.8.0_77/jre
[2017-09-22 10:46:23,858] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Java Version     : 1.8.0_77
[2017-09-22 10:46:23,859] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Java VM          : Java HotSpot(TM) 64-Bit Server VM 25.77-b03,Oracle Corporation
[2017-09-22 10:46:23,859] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Carbon Home      : /home/chamilad/dev/QSP/Gartner-2017/wso2ei-6.1.1/wso2/business-process
[2017-09-22 10:46:23,859] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  Java Temp Dir    : /home/chamilad/dev/QSP/Gartner-2017/wso2ei-6.1.1/wso2/business-process/tmp
[2017-09-22 10:46:23,859] [EI-Business-Process]  INFO {org.wso2.carbon.core.internal.CarbonCoreActivator} -  User             : chamilad, en-US, America/New_York
[2017-09-22 10:46:26,311] [EI-Business-Process]  INFO {org.wso2.carbon.registry.core.jdbc.EmbeddedRegistryService} -  Configured Registry in 158ms
...
...
```

## Containers
Tools like Docker, K8S enable methods to pass environment variables when spawning Containers.

```bash
docker run -e WSO2_SERVER_RUNTIME="business-process" -it wso2ei:6.1.1
```

## Virtual Machines
Platforms like AWS EC2 provide methods to pass environment variables when spawning instances as `user-data`.

![AWS Launch Instance Screen](img/aws-user-data.png "AWS Launch Instance Screen")
