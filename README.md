# WSO2 EI 6.1.1 Bootstrap Script
This script configures different runtimes in WSO2 Enterprise Integrator 6.1.1.

![WSO2 EI Minimum Deployment](img/pattern.png "WSO2 EI Minimum Deployment")

## Supported Runtimes
1. WSO2 EI Integrator
2. WSO2 EI BPS

## How To
The configuration options are expected to be set as environment variables.

1. WSO2_SERVER_RUNTIME - The runtime id of the server. Currently supports either "business-process" or "integrator"
2. WSO2_CARBON_HOME - The location where the WSO2 Server is located
3. WSO2_HOSTNAME - The hostname to be used in the WSO2 servers
4. WSO2_DB_HOSTNAME - The hostname of the DB server
5. WSO2_DB_PORT - The port at which the DB server is operating
6. WSO2_DB_NAME - The DB name
7. WSO2_DB_PROTOCOL - The JDBC URL Protocol to use. This would use a default value of "mysql"
8. WSO2_DB_DRIVER_NAME - The JDBC driver name to be used in the datasources. This would use a default value of "com.mysql.jdbc.Driver"
9. WSO2_DB_USERNAME - The username to access the DB
10. WSO2_DB_PASSWORD - The password to access the DB
11. WSO2_SERVER_ARGS - Arguments to pass to the WSO2 Server starter script

### Database configuration
This script expects the database to be already created and populated with the initial source scripts. If this cannot be done, the option to setup databases automatically can be followed. For this, pass the `-Dsetup` system property to the WSO2 Server starter script.

```bash
cd $CARBON_HOME/bin
./business-process.sh -Dsetup
```

If this system property is passed, WSO2 Carbon will execute the relevant backup scripts on top of the databases specified. However, this would not always result in expected state, especially if the databases specified are not empty.

`WSO2_SERVER_ARGS` can be used to pass `-Dsetup` argument into the script.

```bash
export WSO2_SERVER_ARGS="-Dsetup"
```

> NOTE: Please note that this is not recommended to be used on a production environment, for which the next section should be followed to set up the databases.

#### Setting up the database
For each runtime, the initial database scripts are shipped with the product. These would be inside `CARBON_HOME/wso2/RUNTIME/dbscripts` folder (except for the Integration runtime in which the database scripts are in `CARBON_HOME/dbscripts`). After creating the database, the relevant `*.sql` files can be sourced in. How these should be restored differ based on the type of the RDBMS.

```bash
# Make a copy of the provided sample configuration file conf.sh.sample as conf.sh.
cp conf.sh.sample conf.sh
# Add the properites as needed. A sample set of values can be as follows.
# export WSO2_SERVER_RUNTIME="business-process"
# export WSO2_CARBON_HOME="/home/chamilad/dev/wso2ei-6.1.1"
# export WSO2_HOSTNAME="localhost"
# export WSO2_DB_HOSTNAME="localhost"
# export WSO2_DB_PORT="3306"
# export WSO2_DB_NAME="WSO2BPS_DB"
# export WSO2_DB_DRIVER_NAME=""
# export WSO2_DB_PROTOCOL=""
# export WSO2_DB_USERNAME="root"
# export WSO2_DB_PASSWORD="toor"
# export WSO2_SERVER_ARGS="-Dsetup"
vi conf.sh
# Source the conf.sh.
source conf.sh
# Run setup.sh file. This would start the WSO2 Server at the above specified location.
bash setup.sh
```

## Copying Files
Create the directory structure inside the `files/<runtime>` folder, as the file should be copied to the destination, relative to `CARBON_HOME`. For example, if the MySQL JDBC driver should be copied inside `$CARBON_HOME/lib` folder for the business-process runtim, then the folder structure inside `files` folder should be as follows.

```
files
├── business-process
│   ├── .gitkeep
│   └── lib
|     └──mysql-connector-java-5.1.39-bin.jar
├── common
│   └── .gitkeep
└── integrator
    └── .gitkeep
```

Files copied in the above manner in to the `common` folder will be copied into every runtime. If files exist with the same file name in both the `common` folder and the runtime specific folder, the file in the runtime specific folder would be used.

## Containers
Tools like Docker, K8S enable methods to pass environment variables when spawning Containers.

```bash
docker run -e WSO2_SERVER_RUNTIME="business-process" -it wso2ei:6.1.1
```

## Virtual Machines
Platforms like AWS EC2 provide methods to pass environment variables when spawning instances as `user-data`.

![AWS Launch Instance Screen](img/aws-user-data.png "AWS Launch Instance Screen")
