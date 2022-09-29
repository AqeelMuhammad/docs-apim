# Upgrading API Manager from 2.6.0 to 3.1.0

The following information describes how to upgrade your API Manager server **from APIM 2.6.0 to 3.1.0**.

!!! note
    Before you follow this section, see [Upgrading Process]({{base_path}}/install-and-setup/upgrading-wso2-api-manager/upgrading-process) for more information.

!!! attention "Before you Begin"
    This release is a WUM-only release. This means that there are no manual patches. Any further fixes or latest updates for this release can be updated through the WSO2 Update Manager (WUM).

    - **If you are upgrading to this version, in order to use this version in your production environment**, use the WSO2 Update Manager and get the latest available updates for WSO2 API Manager 3.1.0. For more information on how to do this, see [Updating API Manager]({{base_path}}/administer/updating-wso2-api-manager/#wso2-update-manager-wum).

!!! note "If you are using PostgreSQL"
    The DB user needs to have superuser role to run the migration client and the relevant scripts
    ```
    ALTER USER <user> WITH SUPERUSER;
    ```
!!! note "If you are using Oracle"
    Commit the changes after running the scripts given below.
    
Follow the instructions below to upgrade your WSO2 API Manager server **from WSO2 API-M 2.6.0 to 3.1.0**.

### Preparing for Migration
#### Disable versioning in the registry configuration

If there are frequently updating registry properties, having the versioning enabled for registry resources in the registry can lead to unnecessary growth in the registry related tables in the database. To avoid this, versioning has been disabled by default in API Manager 3.1.0.

Therefore, if registry versioning was enabled in WSO2 API-M 2.6.0 setup, it is **required** run the below scripts against **the database that is used by the registry**.

!!! note "NOTE"
    Alternatively, it is possible to turn on registry versioning in API Manager 3.1.0 and continue. But this is
    highly **NOT RECOMMENDED** and these configurations should only be changed once.

!!! info "Verifying registry versioning turned on in your current API-M and running the scripts"
    Open the `registry.xml` file in the `<OLD_API-M_HOME>/repository/conf` directory.
    Check whether `versioningProperties`, `versioningComments`, `versioningTags` and `versioningRatings` configurations are true.
    
    ```
    <staticConfiguration>
        <versioningProperties>true</versioningProperties>
        <versioningComments>true</versioningComments>
        <versioningTags>true</versioningTags>
        <versioningRatings>true</versioningRatings>
    </staticConfiguration>
    ```
    
    !!! warning
        If the above configurations are already set as `false` you should not run the below scripts.
    
    From API-M 3.0.0 version onwards, those configurations are set to false by-default and since these configurations are now getting changed from old setup to new setup, you need to remove the versioning details from the database in order for the registry resources to work properly. For that, choose the relevant DB type and run the script against the DB that the registry resides in, to remove the registry versioning details.   
    ??? info "DB Scripts"
        ```tab="DB2"
        -- Update the REG_PATH_ID column mapped with the REG_RESOURCE table --

        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION)
        /
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION)
        /

        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION)
        /
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION)
        /

        -- Delete versioned tags, were the PATH_ID will be null for older versions --

        delete from REG_RESOURCE_PROPERTY where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_RATING where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_TAG where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_COMMENT where REG_PATH_ID is NULL
        /
        delete from REG_PROPERTY where REG_ID NOT IN (select REG_PROPERTY_ID from REG_RESOURCE_PROPERTY)
        /
        delete from REG_TAG where REG_ID NOT IN (select REG_TAG_ID from REG_RESOURCE_TAG)
        /
        delete from REG_COMMENT where REG_ID NOT IN (select REG_COMMENT_ID from REG_RESOURCE_COMMENT)
        /
        delete from REG_RATING where REG_ID NOT IN (select REG_RATING_ID from REG_RESOURCE_RATING)
        /

        -- Update the REG_PATH_NAME column mapped with the REG_RESOURCE table --

        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION)
        /
        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION)
        /
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION)
        /
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION)
        /

        ```
    
        ```tab="MSSQL"
        -- Update the REG_PATH_ID column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);
        
        UPDATE REG_RESOURCE_COMMENT SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);
        
        UPDATE REG_RESOURCE_PROPERTY SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);
        
        UPDATE REG_RESOURCE_RATING SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);
        
        -- Delete versioned tags, were the PATH_ID will be null for older versions --
        delete from REG_RESOURCE_PROPERTY where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_RATING where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_TAG where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_COMMENT where REG_PATH_ID is NULL;
        
        delete from REG_PROPERTY where REG_ID NOT IN (select REG_PROPERTY_ID from REG_RESOURCE_PROPERTY);
        
        delete from REG_TAG where REG_ID NOT IN (select REG_TAG_ID from REG_RESOURCE_TAG);
        
        delete from REG_COMMENT where REG_ID NOT IN (select REG_COMMENT_ID from REG_RESOURCE_COMMENT);
        
        delete from REG_RATING where REG_ID NOT IN (select REG_RATING_ID from REG_RESOURCE_RATING);
        
        -- Update the REG_PATH_NAME column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);
        
        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);
        
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);
        
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);  
        ```

        ```tab="MySQL"
        -- Update the REG_PATH_ID column mapped with the REG_RESOURCE table --

        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);

        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);

        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);

        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);


        -- Delete versioned tags, were the PATH_ID will be null for older versions --

        delete from REG_RESOURCE_PROPERTY where REG_PATH_ID is NULL;

        delete from REG_RESOURCE_RATING where REG_PATH_ID is NULL;

        delete from REG_RESOURCE_TAG where REG_PATH_ID is NULL;

        delete from REG_RESOURCE_COMMENT where REG_PATH_ID is NULL;

        delete from REG_PROPERTY where REG_ID NOT IN (select REG_PROPERTY_ID from REG_RESOURCE_PROPERTY);

        delete from REG_TAG where REG_ID NOT IN (select REG_TAG_ID from REG_RESOURCE_TAG);

        delete from REG_COMMENT where REG_ID NOT IN (select REG_COMMENT_ID from REG_RESOURCE_COMMENT);

        delete from REG_RATING where REG_ID NOT IN (select REG_RATING_ID from REG_RESOURCE_RATING);

        -- Update the REG_PATH_NAME column mapped with the REG_RESOURCE table --

        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);

        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);

        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);

        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);
        ```
    
        ```tab="Oracle"
        -- Update the REG_PATH_ID column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION)
        /
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION)
        /
        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION)
        /
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_PATH_ID=(SELECT REG_RESOURCE.REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION)
        /
        
        -- Delete versioned tags, were the PATH_ID will be null for older versions --
        delete from REG_RESOURCE_PROPERTY where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_RATING where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_TAG where REG_PATH_ID is NULL
        /
        delete from REG_RESOURCE_COMMENT where REG_PATH_ID is NULL
        /
        delete from REG_PROPERTY where REG_ID NOT IN (select REG_PROPERTY_ID from REG_RESOURCE_PROPERTY)
        /
        delete from REG_TAG where REG_ID NOT IN (select REG_TAG_ID from REG_RESOURCE_TAG)
        /
        delete from REG_COMMENT where REG_ID NOT IN (select REG_COMMENT_ID from REG_RESOURCE_COMMENT)
        /
        delete from REG_RATING where REG_ID NOT IN (select REG_RATING_ID from REG_RESOURCE_RATING)
        /
        
        -- Update the REG_PATH_NAME column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_TAG.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION)
        /
        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_PROPERTY.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION)
        /
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_COMMENT.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION)
        /
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_RATING.REG_RESOURCE_NAME=(SELECT REG_RESOURCE.REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION)
        /
        COMMIT;
        /
        ```
        
        ```tab="PostgreSQL"
        -- Update the REG_PATH_ID column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);
        
        UPDATE REG_RESOURCE_COMMENT SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);
        
        UPDATE REG_RESOURCE_PROPERTY SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);
        
        UPDATE REG_RESOURCE_RATING SET REG_PATH_ID=(SELECT REG_PATH_ID FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);
        
        -- Delete versioned tags, were the PATH_ID will be null for older versions --
        delete from REG_RESOURCE_PROPERTY where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_RATING where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_TAG where REG_PATH_ID is NULL;
        
        delete from REG_RESOURCE_COMMENT where REG_PATH_ID is NULL;
        
        delete from REG_PROPERTY where REG_ID NOT IN (select REG_PROPERTY_ID from REG_RESOURCE_PROPERTY);
        
        delete from REG_TAG where REG_ID NOT IN (select REG_TAG_ID from REG_RESOURCE_TAG);
        
        delete from REG_COMMENT where REG_ID NOT IN (select REG_COMMENT_ID from REG_RESOURCE_COMMENT);
        
        delete from REG_RATING where REG_ID NOT IN (select REG_RATING_ID from REG_RESOURCE_RATING);
        
        -- Update the REG_PATH_NAME column mapped with the REG_RESOURCE table --
        UPDATE REG_RESOURCE_TAG SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_TAG.REG_VERSION);
        
        UPDATE REG_RESOURCE_PROPERTY SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_PROPERTY.REG_VERSION);
        
        UPDATE REG_RESOURCE_COMMENT SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_COMMENT.REG_VERSION);
        
        UPDATE REG_RESOURCE_RATING SET REG_RESOURCE_NAME=(SELECT REG_NAME FROM REG_RESOURCE WHERE REG_RESOURCE.REG_VERSION=REG_RESOURCE_RATING.REG_VERSION);
        ```
    
!!! warning "Not recommended"
    If you decide to proceed with registry resource versioning enabled, Add the following configuration to the `<NEW_API-M_HOME>/repository/conf/deployment.toml` file of new WSO2 API Manager. 
    
    ```
    [registry.static_configuration]
    enable=true
    ```
    
    !!! note "NOTE"
        Changing these configurations should only be done before the initial API-M Server startup. If changes are done after the initial startup, the registry resource created previously will not be available.

-   [Step 1 - Migrate the API Manager configurations](#step-1-migrate-the-api-manager-configurations)
-   [Step 2 - Upgrade API Manager to 3.1.0](#step-2-upgrade-api-manager-to-310)
-   [Step 3 - Optionally, migrate the configurations for WSO2 API-M Analytics](#step-3-optionally-migrate-the-configurations-for-wso2-api-m-analytics)
-   [Step 4 - Restart the WSO2 API-M 3.1.0 server](#step-4-restart-the-wso2-api-m-310-server)

### Step 1 - Migrate the API Manager configurations

!!! warning
    As the configuration model has been changed, now all the configurations need to be done via a single file (deployment.toml). Therefore, do not copy all the configuration files from the current version of WSO2 API Manager to the new one. Instead, redo the configuration changes in the new configuration file. For more information, see [Configuration Catalog]({{base_path}}/reference/config-catalog).

Follow the instructions below to move all the existing API Manager configurations from the current environment to the new one.

1.  Back up all databases in your API Manager instances along with the Synapse configurations of all the tenants and the super tenant.

    -   The Synapse configurations of the super tenant are in the `<OLD_API-M_HOME>/repository/deployment/server/synapse-configs/default` directory.

    -   The Synapse configurations of tenants are in the `<OLD_API-M_HOME>/repository/tenants` directory.

    -   If you use a **clustered/distributed API Manager setup**, back up the available configurations in the **API Gateway** node.

2.  Download [WUM updated](https://docs.wso2.com/display/updates/Getting+Started) pack for [WSO2 API Manager 3.1.0](http://wso2.com/api-management/).

3.  Open the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file and provide the datasource configurations for the following databases.

    -   User Store
    -   Registry database/s
    -   API Manager databases

    !!! note
        In API-M 3.x versions, a combined SHARED_DB has been introduced to keep both the user related data (`WSO2UM_DB`) and the registry data (`WSO2REG_DB`). If you have used a separate DBs for user management and registry in the previous version, you need to configure the `WSO2REG_DB` and `WSO2UM_DB` databases separately in API-M 3.1.0 to avoid any issues.

    - Datasource configurations in the previous versions could be mapped to the new configuration model as follows:
        
        - **SHARED_DB** should point to the previous API-M version's `WSO2REG_DB`.
        - **APIM_DB** should point to the previous API-M version's `WSO2AM_DB`.
        - **CONFIG** should point to the previous API-M version's `WSO2CONFIG_DB`.
        - **USER** should point to the previous API-M version's ` WSO2USER_DB`.
     
    The following example shows how you can configure the MySQL database configurations.

    ```
    [database.apim_db]
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/am_db"
    username = "username"
    password = "password"

    [database.shared_db]
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/reg_db"
    username = "username"
    password = "password"
    ```

    Optionally, if you have configured a separate user management database in the previous API-M version, add a new entry as shown below in the `deployment.toml` file.

    ```
    [database.user]
    type = "mysql"
    url = "jdbc:mysql://localhost:3306/um_db"
    username = "username"
    password = "password"
    ```

    !!! note
        If you have configured WSO2CONFIG_DB in the previous API-M version, add a new entry to the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` as below.

        ```
        [database.config]
        type = "mysql"
        url = "jdbc:mysql://localhost:3306/config_db"
        username = "username"
        password = "password"
        ```

    !!! attention "If you are using another DB type"
        If you are using another DB type other than **MySQL** or **Oracle**, when defining the DB related configurations in the `deployment.toml` file, you need to add the `driver` and `validationQuery` parameters additionally as given below.

        ```tab="MSSQL"
        [database.apim_db]
        type = "mssql"
        url = "jdbc:sqlserver://localhost:1433;databaseName=mig_am_db;SendStringParametersAsUnicode=false"
        username = "username"
        password = "password"
        driver = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
        validationQuery = "SELECT 1"
        ```

        ```tab="PostgreSQL"
        [database.apim_db]
        type = "postgre"
        url = "jdbc:postgresql://localhost:5432/mig_am_db"
        username = "username"
        password = "password"
        driver = "org.postgresql.Driver"
        validationQuery = "SELECT 1"
        ```

        ```tab="Oracle"
        [database.apim_db]
        type = "oracle"
        url = "jdbc:oracle:thin:@localhost:1521/mig_am_db"
        username = "username"
        password = "password"
        driver = "oracle.jdbc.driver.OracleDriver"
        validationQuery = "SELECT 1 FROM DUAL"
        ```

        ```tab="DB2"
        [database.apim_db]
        type = "db2"
        url = "jdbc:db2://localhost:50000/mig_am_db"
        username = "username"
        password = "password"
        driver = "com.ibm.db2.jcc.DB2Driver"
        validationQuery = "SELECT 1 FROM SYSIBM.SYSDUMMY1"
        ```

4. If you have used a separate DB for user management in your previous setup and you have defined the `[database.user]` as mentioned in the above step, then you need to update the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file as follows, to point to the correct database for user management purposes. **Note** that the data_source name cannot be changed and it should be **WSO2USER_DB**.

    ```
    [realm_manager]
    data_source = "WSO2USER_DB"
    ```

5.  Copy the relevant JDBC driver to the `<API-M_3.1.0_HOME>/repository/components/lib` folder.

    !!! info
        In API-M 3.1.0, you do not need to configure the registry configurations as you did in the `<OLD_API-M_HOME>/repository/conf/registry.xml` file and the user database configurations as you did in in the `<OLD_API-M_HOME>/repository/conf/user-mgt.xml` file, as those configurations have been handled internally.

6.  Move all your Synapse configurations to API-M 3.1.0 pack.
    -   Move your Synapse super tenant configurations.
        Copy the content in the `<OLD_API-M_HOME>/repository/deployment/server/synapse-configs/default` directory and replace the content in the `<API-M_3.1.0_HOME>/repository/deployment/server/synapse-configs/default` directory with the copied content.
    -   Move all your tenant Synapse configurations.
        Copy the content in the `<OLD_API-M_HOME>/repository/tenants` directory and replace the content in the `<API-M_3.1.0_HOME>/repository/tenants` directory with the copied content.

    !!! warning
        When moving the Synapse configurations, **do not replace** the following set of files as they contain some modifications in API-M 3.1.0.

        -   /api/\_RevokeAPI_.xml
        -   /sequences/\_cors_request_handler_.xml
        -   /sequences/main.xml
        -   /proxy-services/WorkflowCallbackService.xml

    !!! attention 
        If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Gateway** node.

7.  Move all your Execution plans from the `<API-M_2.6.0_HOME>/repository/deployment/server/executionplans` directory to the `<API-M_3.1.0_HOME>/repository/deployment/server/executionplans` directory.

    !!! note
        If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Traffic Manager** node.

8.  If you manually added any custom OSGI bundles to the `<API-M_2.6.0_HOME>/repository/components/dropins` directory, copy those to the `<API-M_3.1.0_HOME>/repository/components/dropins` directory. 

9.  If you manually added any JAR files to the `<API-M_2.6.0_HOME>/repository/components/lib` directory, copy those and paste them in the `<API-M_3.1.0_HOME>/repository/components/lib` directory.

10. WSO2 API Manager 3.1.0 has been upgraded to log4j2 (from log4j). You will notice that there is a `log4j2.properties` file in the `<API-M_3.1.0_HOME>/repository/conf/` directory instead of the `log4j.properties` file. Follow the instructions in the [Upgrading to Log4j2]({{base_path}}/install-and-setup/upgrading-wso2-api-manager/upgrading-to-log4j2) document to migrate your existing `log4j.properties` file to `log4j2.properties` file.

    !!! warning
        Taking the `log4j.properties` file from your old WSO2 API-M Server and adding it to WSO2 API-M Server 3.1.0 will no longer work. For information on how to add a log appender or a logger to the `log4j2.properties` file, see [Upgrading to Log4j2]({{base_path}}/install-and-setup/upgrading-wso2-api-manager/upgrading-to-log4j2).

    !!! note
        Log4j2 has hot deployment support, and **Managing Logs** section has been removed from the Management Console. You can now use the `log4j2.properties` file to modify the logging configurations without restarting the server.

### Step 2 - Upgrade API Manager to 3.1.0

1.  Stop all WSO2 API Manager server instances that are running.

2.  Make sure you backed up all the databases and Synapse configurations as instructed in [step 1](#step-1-migrate-the-api-manager-configurations) of the previous section.

3.  Upgrade the WSO2 API Manager database from version 2.6.0 to version 3.1.0 by executing the relevant database script, from the scripts that are provided below, on the `WSO2AM_DB` database.

    ??? info "DB Scripts"
        ```tab="DB2"
        CREATE TABLE AM_SYSTEM_APPS (
            ID INTEGER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
            NAME VARCHAR(50) NOT NULL,
            CONSUMER_KEY VARCHAR(512) NOT NULL,
            CONSUMER_SECRET VARCHAR(512) NOT NULL,
            CREATED_TIME TIMESTAMP(0) NOT NULL DEFAULT CURRENT_TIMESTAMP,
            UNIQUE (NAME),
            UNIQUE (CONSUMER_KEY),
            PRIMARY KEY (ID)
        )
        /

        CREATE TABLE AM_API_CLIENT_CERTIFICATE (
            TENANT_ID INT NOT NULL,
            ALIAS VARCHAR(45) NOT NULL,
            API_ID INTEGER NOT NULL,
            CERTIFICATE BLOB NOT NULL,
            REMOVED SMALLINT NOT NULL DEFAULT 0,
            TIER_NAME VARCHAR (512),
            FOREIGN KEY (API_ID) REFERENCES AM_API (API_ID) ON DELETE CASCADE,
            PRIMARY KEY (ALIAS, TENANT_ID, REMOVED)
        )
        /

        ALTER TABLE AM_POLICY_SUBSCRIPTION 
        ADD MONETIZATION_PLAN VARCHAR(25) DEFAULT NULL
        ADD FIXED_RATE VARCHAR(15) DEFAULT NULL
        ADD BILLING_CYCLE VARCHAR(15) DEFAULT NULL 
        ADD PRICE_PER_REQUEST VARCHAR(15) DEFAULT NULL 
        ADD CURRENCY VARCHAR(15) DEFAULT NULL
        /

        CREATE TABLE AM_MONETIZATION_USAGE (
            ID VARCHAR(100) NOT NULL,
            STATE VARCHAR(50) NOT NULL,
            STATUS VARCHAR(50) NOT NULL,
            STARTED_TIME VARCHAR(50) NOT NULL,
            PUBLISHED_TIME VARCHAR(50) NOT NULL,
            PRIMARY KEY(ID)
        )/

        ALTER TABLE AM_API_COMMENTS DROP PRIMARY KEY;
        ALTER TABLE AM_API_COMMENTS DROP COMMENT_ID;
        ALTER TABLE AM_API_COMMENTS ADD COMMENT_ID VARCHAR(255) NOT NULL DEFAULT '0';
        CALL ADMIN_CMD('REORG table AM_API_COMMENTS');
        UPDATE AM_API_COMMENTS SET COMMENT_ID=(hex(GENERATE_UNIQUE())) WHERE COMMENT_ID='0';
        ALTER TABLE AM_API_COMMENTS PRIMARY KEY (COMMENT_ID);

        ALTER TABLE AM_API_RATINGS DROP PRIMARY KEY;
        ALTER TABLE AM_API_RATINGS DROP RATING_ID;
        ALTER TABLE AM_API_RATINGS ADD RATING_ID VARCHAR(255) NOT NULL DEFAULT '0';
        CALL ADMIN_CMD('REORG table AM_API_RATINGS');
        UPDATE AM_API_RATINGS SET RATING_ID=(hex(GENERATE_UNIQUE())) WHERE RATING_ID='0';
        ALTER TABLE AM_API_RATINGS PRIMARY KEY (RATING_ID);

        CREATE TABLE IF NOT EXISTS AM_NOTIFICATION_SUBSCRIBER (
            UUID VARCHAR(255) NOT NULL,
            CATEGORY VARCHAR(255) NOT NULL,
            NOTIFICATION_METHOD VARCHAR(255) NOT NULL,
            SUBSCRIBER_ADDRESS VARCHAR(255) NOT NULL,
            PRIMARY KEY(UUID, SUBSCRIBER_ADDRESS)
        )
        /

        ALTER TABLE AM_EXTERNAL_STORES
        ADD LAST_UPDATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        /

        ALTER TABLE AM_API
        ADD API_TYPE VARCHAR(10) NULL DEFAULT NULL
        /

        CREATE TABLE AM_API_PRODUCT_MAPPING (
        API_PRODUCT_MAPPING_ID INTEGER GENERATED ALWAYS AS IDENTITY (START WITH 1 INCREMENT BY 1),
        API_ID INTEGER,
        URL_MAPPING_ID INTEGER,
        FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE,
        FOREIGN KEY (URL_MAPPING_ID) REFERENCES AM_API_URL_MAPPING(URL_MAPPING_ID) ON DELETE CASCADE,
        PRIMARY KEY(API_PRODUCT_MAPPING_ID)
        )
        /

        CREATE TABLE AM_REVOKED_JWT (
        UUID VARCHAR(255) NOT NULL,
        SIGNATURE VARCHAR(2048) NOT NULL,
        EXPIRY_TIMESTAMP BIGINT NOT NULL,
        TENANT_ID INTEGER DEFAULT -1,
        TOKEN_TYPE VARCHAR(15) DEFAULT 'DEFAULT',
        TIME_CREATED TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (UUID)
        )
        /

        -- UMA tables --
        CREATE TABLE IDN_UMA_RESOURCE (
        ID                  INTEGER   NOT NULL,
        RESOURCE_ID         VARCHAR(255),
        RESOURCE_NAME       VARCHAR(255),
        TIME_CREATED        TIMESTAMP NOT NULL,
        RESOURCE_OWNER_NAME VARCHAR(255),
        CLIENT_ID           VARCHAR(255),
        TENANT_ID           INTEGER DEFAULT -1234,
        USER_DOMAIN         VARCHAR(50),
        PRIMARY KEY (ID)
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_RESOURCE_TRIG NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_RESOURCE
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_RESOURCE_SEQ);
        END
        /

        CREATE INDEX IDX_RID ON IDN_UMA_RESOURCE (RESOURCE_ID)
        /

        CREATE INDEX IDX_USER ON IDN_UMA_RESOURCE (RESOURCE_OWNER_NAME, USER_DOMAIN)
        /

        CREATE TABLE IDN_UMA_RESOURCE_META_DATA (
            ID INTEGER NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            PROPERTY_KEY VARCHAR(40),
            PROPERTY_VALUE VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_META_DATA_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_RESOURCE_META_DATA_TRIG NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_RESOURCE_META_DATA
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_RESOURCE_META_DATA_SEQ);
        END
        /

        CREATE TABLE IDN_UMA_RESOURCE_SCOPE (
            ID INTEGER NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            SCOPE_NAME VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_SCOPE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_RESOURCE_SCOPE_TRIG  NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_RESOURCE_SCOPE
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_RESOURCE_SCOPE_SEQ);
        END
        /

        CREATE INDEX IDX_RS ON IDN_UMA_RESOURCE_SCOPE (SCOPE_NAME)
        /

        CREATE TABLE IDN_UMA_PERMISSION_TICKET (
            ID INTEGER NOT NULL,
            PT VARCHAR(255) NOT NULL,
            TIME_CREATED TIMESTAMP NOT NULL,
            EXPIRY_TIME TIMESTAMP NOT NULL,
            TICKET_STATE VARCHAR(25) DEFAULT 'ACTIVE',
            TENANT_ID INTEGER DEFAULT -1234,
            PRIMARY KEY (ID)
        )
        /

        CREATE SEQUENCE IDN_UMA_PERMISSION_TICKET_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_PERMISSION_TICKET_TRIG NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_PERMISSION_TICKET
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_PERMISSION_TICKET_SEQ);
        END
        /

        CREATE INDEX IDX_PT ON IDN_UMA_PERMISSION_TICKET (PT)
        /

        CREATE TABLE IDN_UMA_PT_RESOURCE (
            ID INTEGER NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_ID) REFERENCES IDN_UMA_PERMISSION_TICKET (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_PT_RESOURCE_TRIG NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_PT_RESOURCE
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_PT_RESOURCE_SEQ);
        END
        /

        CREATE TABLE IDN_UMA_PT_RESOURCE_SCOPE (
            ID INTEGER NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_SCOPE_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_PT_RESOURCE (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_SCOPE_ID) REFERENCES IDN_UMA_RESOURCE_SCOPE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SCOPE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE TRIGGER IDN_UMA_PT_RESOURCE_SCOPE_TRIG NO CASCADE
        BEFORE INSERT
        ON IDN_UMA_PT_RESOURCE_SCOPE
        REFERENCING NEW AS NEW
        FOR EACH ROW MODE DB2SQL
        BEGIN ATOMIC
            SET (NEW.ID) = (NEXTVAL FOR IDN_UMA_PT_RESOURCE_SCOPE_SEQ);
        END
        /

        CREATE TABLE AM_API_CATEGORIES (
            UUID VARCHAR(50) NOT NULL,
            NAME VARCHAR(255) NOT NULL,
            DESCRIPTION VARCHAR(1024),
            TENANT_ID INTEGER NOT NULL DEFAULT -1,
            UNIQUE (NAME,TENANT_ID),
            PRIMARY KEY (UUID)
        ) /

        ALTER TABLE AM_SYSTEM_APPS
        ADD TENANT_DOMAIN VARCHAR(255) DEFAULT 'carbon.super'
        /

        CREATE TABLE AM_USER (
            USER_ID VARCHAR(255) NOT NULL,
            USER_NAME VARCHAR(255) NOT NULL,
            PRIMARY KEY(USER_ID)
        ) /

        CREATE TABLE AM_SECURITY_AUDIT_UUID_MAPPING (
            API_ID INTEGER NOT NULL,
            AUDIT_UUID VARCHAR(255) NOT NULL,
            PRIMARY KEY (API_ID),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID)
        ) /

        DELETE FROM IDN_OAUTH2_SCOPE_BINDING WHERE SCOPE_BINDING IS NULL OR SCOPE_BINDING = ''  / 
        ```

        ```tab="MSSQL"
        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'AM_SYSTEM_APPS') AND TYPE IN (N'U'))
        CREATE TABLE AM_SYSTEM_APPS (
            ID INTEGER IDENTITY,
            NAME VARCHAR(50) NOT NULL,
            CONSUMER_KEY VARCHAR(512) NOT NULL,
            CONSUMER_SECRET VARCHAR(512) NOT NULL,
            CREATED_TIME DATETIME2(6) DEFAULT CURRENT_TIMESTAMP,
            UNIQUE (NAME),
            UNIQUE (CONSUMER_KEY),
            PRIMARY KEY (ID)
        );

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_API_CLIENT_CERTIFICATE]') AND TYPE IN (N'U'))
        CREATE TABLE AM_API_CLIENT_CERTIFICATE (
            TENANT_ID INTEGER NOT NULL,
            ALIAS VARCHAR(45) NOT NULL,
            API_ID INTEGER NOT NULL,
            CERTIFICATE VARBINARY(MAX) NOT NULL,
            REMOVED BIT NOT NULL DEFAULT 0,
            TIER_NAME VARCHAR(512),
            PRIMARY KEY (ALIAS, TENANT_ID, REMOVED),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE
        );

        ALTER TABLE AM_POLICY_SUBSCRIPTION ADD
        MONETIZATION_PLAN VARCHAR(25) NULL DEFAULT NULL,
        FIXED_RATE VARCHAR(15) NULL DEFAULT NULL, 
        BILLING_CYCLE VARCHAR(15) NULL DEFAULT NULL, 
        PRICE_PER_REQUEST VARCHAR(15) NULL DEFAULT NULL, 
        CURRENCY VARCHAR(15) NULL DEFAULT NULL
        ;

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_MONETIZATION_USAGE]') AND TYPE IN (N'U'))

        CREATE TABLE AM_MONETIZATION_USAGE (
            ID VARCHAR(100) NOT NULL,
            STATE VARCHAR(50) NOT NULL,
            STATUS VARCHAR(50) NOT NULL,
            STARTED_TIME VARCHAR(50) NOT NULL,
            PUBLISHED_TIME VARCHAR(50) NOT NULL,
            PRIMARY KEY(ID)
        );

        DECLARE @con_com as VARCHAR(8000);
        SET @con_com = (SELECT name from sys.objects where parent_object_id=object_id('AM_API_COMMENTS') AND type='PK');
        EXEC('ALTER TABLE AM_API_COMMENTS
        drop CONSTRAINT ' + @con_com);
        ALTER TABLE AM_API_COMMENTS
        DROP COLUMN COMMENT_ID;
        ALTER TABLE AM_API_COMMENTS
        ADD COMMENT_ID VARCHAR(255) NOT NULL DEFAULT NEWID();
        ALTER TABLE AM_API_COMMENTS
        ADD PRIMARY KEY (COMMENT_ID);

        DECLARE @con_rat as VARCHAR(8000);
        SET @con_rat = (SELECT name from sys.objects where parent_object_id=object_id('AM_API_RATINGS') AND type='PK');
        EXEC('ALTER TABLE AM_API_RATINGS
        drop CONSTRAINT ' + @con_rat);
        ALTER TABLE AM_API_RATINGS
        DROP COLUMN RATING_ID;
        ALTER TABLE AM_API_RATINGS
        ADD RATING_ID VARCHAR(255) NOT NULL DEFAULT NEWID();
        ALTER TABLE AM_API_RATINGS
        ADD PRIMARY KEY (RATING_ID);

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_NOTIFICATION_SUBSCRIBER]') AND TYPE IN (N'U'))
        CREATE TABLE AM_NOTIFICATION_SUBSCRIBER (
            UUID VARCHAR(255),
            CATEGORY VARCHAR(255),
            NOTIFICATION_METHOD VARCHAR(255),
            SUBSCRIBER_ADDRESS VARCHAR(255) NOT NULL,
            PRIMARY KEY(UUID, SUBSCRIBER_ADDRESS)
        );

        ALTER TABLE AM_EXTERNAL_STORES
        ADD LAST_UPDATED_TIME DATETIME DEFAULT GETDATE();

        ALTER TABLE AM_API
            ADD API_TYPE VARCHAR(10) NULL DEFAULT NULL;

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_API_PRODUCT_MAPPING]') AND TYPE IN (N'U'))

        CREATE TABLE AM_API_PRODUCT_MAPPING (
            API_PRODUCT_MAPPING_ID INTEGER IDENTITY(1,1),
            API_ID INTEGER,
            URL_MAPPING_ID INTEGER,
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE,
            FOREIGN KEY (URL_MAPPING_ID) REFERENCES AM_API_URL_MAPPING(URL_MAPPING_ID) ON DELETE CASCADE,
            PRIMARY KEY(API_PRODUCT_MAPPING_ID)
        );

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_REVOKED_JWT]') AND TYPE IN (N'U'))
        CREATE TABLE AM_REVOKED_JWT (
            UUID VARCHAR(255) NOT NULL,
            SIGNATURE VARCHAR(2048) NOT NULL,
            EXPIRY_TIMESTAMP BIGINT NOT NULL,
            TENANT_ID INTEGER DEFAULT -1,
            TOKEN_TYPE VARCHAR(15) DEFAULT 'DEFAULT',
            TIME_CREATED DATETIME DEFAULT GETDATE(),
            PRIMARY KEY (UUID)
        );

        -- UMA tables --
        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_RESOURCE]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_RESOURCE (
        ID INTEGER IDENTITY NOT NULL,
        RESOURCE_ID VARCHAR(255),
        RESOURCE_NAME VARCHAR(255),
        TIME_CREATED DATETIME NOT NULL,
        RESOURCE_OWNER_NAME VARCHAR(255),
        CLIENT_ID VARCHAR(255),
        TENANT_ID INTEGER DEFAULT -1234,
        USER_DOMAIN VARCHAR(50),
        PRIMARY KEY (ID)
        );

        IF NOT EXISTS (SELECT * FROM SYS.indexes WHERE name = 'IDX_RID' and object_id = OBJECT_ID('IDN_UMA_RESOURCE'))
        CREATE INDEX IDX_RID ON IDN_UMA_RESOURCE(RESOURCE_ID);
        
        IF NOT EXISTS (SELECT * FROM SYS.indexes WHERE name = 'IDX_USER' and object_id = OBJECT_ID('IDN_UMA_RESOURCE'))
        CREATE INDEX IDX_USER ON IDN_UMA_RESOURCE(RESOURCE_OWNER_NAME, USER_DOMAIN);

        IF NOT EXISTS ( SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_RESOURCE_META_DATA]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_RESOURCE_META_DATA (
            ID INTEGER IDENTITY NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            PROPERTY_KEY VARCHAR(40),
            PROPERTY_VALUE VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        IF NOT EXISTS ( SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_RESOURCE_SCOPE]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_RESOURCE_SCOPE (
            ID INTEGER IDENTITY NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            SCOPE_NAME VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        IF NOT EXISTS (SELECT * FROM SYS.indexes WHERE name = 'IDX_RS' and object_id = OBJECT_ID('IDN_UMA_RESOURCE_SCOPE'))
        CREATE INDEX IDX_RS ON IDN_UMA_RESOURCE_SCOPE(SCOPE_NAME);

        IF NOT EXISTS ( SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_PERMISSION_TICKET]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_PERMISSION_TICKET (
            ID INTEGER IDENTITY NOT NULL,
            PT VARCHAR(255) NOT NULL,
            TIME_CREATED DATETIME NOT NULL,
            EXPIRY_TIME DATETIME NOT NULL,
            TICKET_STATE VARCHAR(25) DEFAULT 'ACTIVE',
            TENANT_ID INTEGER     DEFAULT -1234,
            PRIMARY KEY (ID)
        );

        IF NOT EXISTS (SELECT * FROM SYS.indexes WHERE name = 'IDX_PT' and object_id = OBJECT_ID('IDN_UMA_PERMISSION_TICKET'))
        CREATE INDEX IDX_PT ON IDN_UMA_PERMISSION_TICKET(PT);

        IF NOT EXISTS ( SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_PT_RESOURCE]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_PT_RESOURCE (
            ID INTEGER IDENTITY NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_ID) REFERENCES IDN_UMA_PERMISSION_TICKET (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_RESOURCE (ID)
        );

        IF NOT EXISTS ( SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[IDN_UMA_PT_RESOURCE_SCOPE]') AND TYPE IN (N'U'))
        CREATE TABLE IDN_UMA_PT_RESOURCE_SCOPE (
            ID INTEGER IDENTITY NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_SCOPE_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_PT_RESOURCE (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_SCOPE_ID) REFERENCES IDN_UMA_RESOURCE_SCOPE (ID)
        );

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_API_CATEGORIES]') AND TYPE IN (N'U'))
        CREATE TABLE AM_API_CATEGORIES (
            UUID VARCHAR(50),
            NAME VARCHAR(255),
            DESCRIPTION VARCHAR(1024),
            TENANT_ID INTEGER DEFAULT -1,
            UNIQUE (NAME,TENANT_ID),
            PRIMARY KEY (UUID)
        );

        ALTER TABLE AM_SYSTEM_APPS
        ADD TENANT_DOMAIN VARCHAR(255) DEFAULT 'carbon.super';

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_USER]') AND TYPE IN (N'U'))
        CREATE TABLE AM_USER (
            USER_ID VARCHAR(255) NOT NULL,
            USER_NAME VARCHAR(255) NOT NULL,
            PRIMARY KEY(USER_ID)
        );

        IF NOT  EXISTS (SELECT * FROM SYS.OBJECTS WHERE OBJECT_ID = OBJECT_ID(N'[DBO].[AM_SECURITY_AUDIT_UUID_MAPPING]') AND TYPE IN (N'U'))
        CREATE TABLE AM_SECURITY_AUDIT_UUID_MAPPING (
            API_ID INTEGER NOT NULL,
            AUDIT_UUID VARCHAR(255) NOT NULL,
            PRIMARY KEY (API_ID),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID)
        );

        DELETE FROM IDN_OAUTH2_SCOPE_BINDING WHERE SCOPE_BINDING IS NULL OR SCOPE_BINDING = '';
        ```

        ```tab="MySQL"
        CREATE TABLE IF NOT EXISTS AM_SYSTEM_APPS (
            ID INTEGER AUTO_INCREMENT,
            NAME VARCHAR(50) NOT NULL,
            CONSUMER_KEY VARCHAR(512) NOT NULL,
            CONSUMER_SECRET VARCHAR(512) NOT NULL,
            CREATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            UNIQUE (NAME),
            UNIQUE (CONSUMER_KEY),
            PRIMARY KEY (ID)
        ) ENGINE=InnoDB;

        CREATE TABLE IF NOT EXISTS AM_API_CLIENT_CERTIFICATE (
            TENANT_ID INT(11) NOT NULL,
            ALIAS VARCHAR(45) NOT NULL,
            API_ID INTEGER NOT NULL,
            CERTIFICATE BLOB NOT NULL,
            REMOVED BOOLEAN NOT NULL DEFAULT 0,
            TIER_NAME VARCHAR (512),
            FOREIGN KEY (API_ID) REFERENCES AM_API (API_ID) ON DELETE CASCADE ON UPDATE CASCADE,
            PRIMARY KEY (ALIAS,TENANT_ID, REMOVED)
        );

        ALTER TABLE AM_POLICY_SUBSCRIPTION 
        ADD MONETIZATION_PLAN VARCHAR(25) NULL DEFAULT NULL, 
        ADD FIXED_RATE VARCHAR(15) NULL DEFAULT NULL, 
        ADD BILLING_CYCLE VARCHAR(15) NULL DEFAULT NULL, 
        ADD PRICE_PER_REQUEST VARCHAR(15) NULL DEFAULT NULL, 
        ADD CURRENCY VARCHAR(15) NULL DEFAULT NULL;

        CREATE TABLE IF NOT EXISTS AM_MONETIZATION_USAGE (
            ID VARCHAR(100) NOT NULL,
            STATE VARCHAR(50) NOT NULL,
            STATUS VARCHAR(50) NOT NULL,
            STARTED_TIME VARCHAR(50) NOT NULL,
            PUBLISHED_TIME VARCHAR(50) NOT NULL,
            PRIMARY KEY(ID)
        ) ENGINE INNODB;

        ALTER TABLE AM_API_COMMENTS
        MODIFY COLUMN COMMENT_ID VARCHAR(255) NOT NULL;

        ALTER TABLE AM_API_RATINGS
        MODIFY COLUMN RATING_ID VARCHAR(255) NOT NULL;

        CREATE TABLE IF NOT EXISTS AM_NOTIFICATION_SUBSCRIBER (
            UUID VARCHAR(255),
            CATEGORY VARCHAR(255),
            NOTIFICATION_METHOD VARCHAR(255),
            SUBSCRIBER_ADDRESS VARCHAR(255) NOT NULL,
            PRIMARY KEY(UUID, SUBSCRIBER_ADDRESS)
        ) ENGINE INNODB;

        ALTER TABLE AM_EXTERNAL_STORES
        ADD LAST_UPDATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

        ALTER TABLE AM_API
        ADD API_TYPE VARCHAR(10) NULL DEFAULT NULL;

        CREATE TABLE IF NOT EXISTS AM_API_PRODUCT_MAPPING (
            API_PRODUCT_MAPPING_ID INTEGER AUTO_INCREMENT,
            API_ID INTEGER,
            URL_MAPPING_ID INTEGER,
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE,
            FOREIGN KEY (URL_MAPPING_ID) REFERENCES AM_API_URL_MAPPING(URL_MAPPING_ID) ON DELETE CASCADE,
            PRIMARY KEY(API_PRODUCT_MAPPING_ID)
        ) ENGINE INNODB;

        CREATE TABLE IF NOT EXISTS AM_REVOKED_JWT (
            UUID VARCHAR(255) NOT NULL,
            SIGNATURE VARCHAR(2048) NOT NULL,
            EXPIRY_TIMESTAMP BIGINT NOT NULL,
            TENANT_ID INTEGER DEFAULT -1,
            TOKEN_TYPE VARCHAR(15) DEFAULT 'DEFAULT',
            TIME_CREATED TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            PRIMARY KEY (UUID)
        ) ENGINE=InnoDB;

        -- UMA tables --
        CREATE TABLE IF NOT EXISTS IDN_UMA_RESOURCE (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            RESOURCE_ID VARCHAR(255),
            RESOURCE_NAME VARCHAR(255),
            TIME_CREATED TIMESTAMP NOT NULL,
            RESOURCE_OWNER_NAME VARCHAR(255),
            CLIENT_ID VARCHAR(255),
            TENANT_ID INTEGER DEFAULT -1234,
            USER_DOMAIN VARCHAR(50),
            PRIMARY KEY (ID)
        );

        DELIMITER $$
        create procedure SKIP_INDEX_IF_EXISTS(indexName varchar(64), tableName varchar(64), tableColumns varchar(255))
        BEGIN
            BEGIN
            DECLARE CONTINUE HANDLER FOR SQLEXCEPTION BEGIN
                END;
            SET @s = CONCAT('CREATE INDEX ', indexName, ' ON ', tableName, tableColumns); PREPARE stmt FROM @s;
                EXECUTE stmt; 
            END;
        END $$
        DELIMITER ;
        
        CALL SKIP_INDEX_IF_EXISTS('IDX_RID', 'IDN_UMA_RESOURCE', '(RESOURCE_ID)');
        
        CALL SKIP_INDEX_IF_EXISTS('IDX_USER', 'IDN_UMA_RESOURCE', '(RESOURCE_OWNER_NAME, USER_DOMAIN)');

        CREATE TABLE IF NOT EXISTS IDN_UMA_RESOURCE_META_DATA (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            PROPERTY_KEY VARCHAR(40),
            PROPERTY_VALUE VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        CREATE TABLE IF NOT EXISTS IDN_UMA_RESOURCE_SCOPE (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            SCOPE_NAME VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        CALL SKIP_INDEX_IF_EXISTS('IDX_RS', 'IDN_UMA_RESOURCE_SCOPE', '(SCOPE_NAME)');

        CREATE TABLE IF NOT EXISTS IDN_UMA_PERMISSION_TICKET (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            PT VARCHAR(255) NOT NULL,
            TIME_CREATED TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            EXPIRY_TIME TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            TICKET_STATE VARCHAR(25) DEFAULT 'ACTIVE',
            TENANT_ID INTEGER DEFAULT -1234,
            PRIMARY KEY (ID)
        );

        CALL SKIP_INDEX_IF_EXISTS('IDX_PT', 'IDN_UMA_PERMISSION_TICKET', '(PT)');

        DROP PROCEDURE IF EXISTS SKIP_INDEX_IF_EXISTS;

        CREATE TABLE IF NOT EXISTS IDN_UMA_PT_RESOURCE (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_ID) REFERENCES IDN_UMA_PERMISSION_TICKET (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        CREATE TABLE IF NOT EXISTS IDN_UMA_PT_RESOURCE_SCOPE (
            ID INTEGER AUTO_INCREMENT NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_SCOPE_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_PT_RESOURCE (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_SCOPE_ID) REFERENCES IDN_UMA_RESOURCE_SCOPE (ID) ON DELETE CASCADE
        );

        CREATE TABLE IF NOT EXISTS AM_API_CATEGORIES (
            UUID VARCHAR(50),
            NAME VARCHAR(255),
            DESCRIPTION VARCHAR(1024),
            TENANT_ID INTEGER DEFAULT -1,
            UNIQUE (NAME,TENANT_ID),
            PRIMARY KEY (UUID)
        ) ENGINE=InnoDB;

        ALTER TABLE AM_SYSTEM_APPS
        ADD TENANT_DOMAIN VARCHAR(255) DEFAULT 'carbon.super';

        ALTER TABLE AM_SYSTEM_APPS
        DROP INDEX NAME;

        CREATE TABLE IF NOT EXISTS AM_USER (
            USER_ID VARCHAR(255) NOT NULL,
            USER_NAME VARCHAR(255) NOT NULL,
            PRIMARY KEY(USER_ID)
        ) ENGINE=InnoDB;

        CREATE TABLE IF NOT EXISTS AM_SECURITY_AUDIT_UUID_MAPPING (
            API_ID INTEGER NOT NULL,
            AUDIT_UUID VARCHAR(255) NOT NULL,
            PRIMARY KEY (API_ID),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID)
        ) ENGINE INNODB;

        DELETE FROM IDN_OAUTH2_SCOPE_BINDING WHERE SCOPE_BINDING IS NULL OR SCOPE_BINDING = '';        
        ```
    
        ```tab="Oracle"
        CREATE TABLE AM_SYSTEM_APPS (
            ID INTEGER,
            NAME VARCHAR2(50) NOT NULL,
            CONSUMER_KEY VARCHAR2(512) NOT NULL,
            CONSUMER_SECRET VARCHAR2(512) NOT NULL,
            CREATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            UNIQUE (NAME),
            UNIQUE (CONSUMER_KEY),
            PRIMARY KEY (ID)
        )
        /

        CREATE SEQUENCE AM_SYSTEM_APP_SEQUENCE START WITH 1 INCREMENT BY 1 NOCACHE
        /
        CREATE OR REPLACE TRIGGER AM_SYSTEM_APPS_TRIG
        BEFORE INSERT
        ON AM_SYSTEM_APPS
        REFERENCING NEW AS NEW
        FOR EACH ROW
            BEGIN
                SELECT AM_SYSTEM_APP_SEQUENCE.nextval INTO :NEW.ID FROM dual;
            END;
        /

        DECLARE table_count NUMBER;
          BEGIN
            table_count := 0;
            SELECT COUNT(1) INTO table_count from user_tables WHERE table_name= 'AM_API_CLIENT_CERTIFICATE';
            IF table_count = 0 THEN
            EXECUTE IMMEDIATE 'CREATE TABLE AM_API_CLIENT_CERTIFICATE (
            TENANT_ID INTEGER NOT NULL,
            ALIAS VARCHAR2(45) NOT NULL,
            API_ID INTEGER NOT NULL,
            CERTIFICATE BLOB NOT NULL,
            REMOVED INTEGER DEFAULT 0 NOT NULL,
            TIER_NAME VARCHAR2 (512),
            FOREIGN KEY (API_ID) REFERENCES AM_API (API_ID) ON DELETE CASCADE,
            PRIMARY KEY (ALIAS, TENANT_ID, REMOVED)
            )';
          END IF;
        END;
        /

        ALTER TABLE AM_POLICY_SUBSCRIPTION ADD (
            MONETIZATION_PLAN VARCHAR(25) DEFAULT NULL NULL, 
            FIXED_RATE VARCHAR(15) DEFAULT NULL NULL, 
            BILLING_CYCLE VARCHAR(15) DEFAULT NULL NULL, 
            PRICE_PER_REQUEST VARCHAR(15) DEFAULT NULL NULL, 
            CURRENCY VARCHAR(15) DEFAULT NULL NULL
        )
        /

        CREATE TABLE AM_MONETIZATION_USAGE (
            ID VARCHAR(100) NOT NULL,
            STATE VARCHAR(50) NOT NULL,
            STATUS VARCHAR(50) NOT NULL,
            STARTED_TIME VARCHAR(50) NOT NULL,
            PUBLISHED_TIME VARCHAR(50) NOT NULL,
            PRIMARY KEY(ID)
        )
        /

        ALTER TABLE AM_API_COMMENTS
            DROP COLUMN COMMENT_ID
        /

        ALTER TABLE AM_API_COMMENTS
            ADD COMMENT_ID VARCHAR(255) DEFAULT (SYS_GUID()) NOT NULL
        /

        ALTER TABLE AM_API_RATINGS
            DROP COLUMN RATING_ID
        /

        ALTER TABLE AM_API_RATINGS
            ADD RATING_ID VARCHAR(255) DEFAULT (SYS_GUID()) NOT NULL
        /

        CREATE TABLE AM_NOTIFICATION_SUBSCRIBER (
            UUID VARCHAR2(255),
            CATEGORY VARCHAR2(255),
            NOTIFICATION_METHOD VARCHAR2(255),
            SUBSCRIBER_ADDRESS VARCHAR2(255) NOT NULL,
            PRIMARY KEY(UUID, SUBSCRIBER_ADDRESS)
        )
        /

        DROP SEQUENCE AM_API_COMMENTS_SEQUENCE
        /

        DROP TRIGGER AM_API_COMMENTS_TRIGGER
        /

        ALTER TABLE AM_EXTERNAL_STORES
        ADD LAST_UPDATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP
        /

        ALTER TABLE AM_API
        ADD API_TYPE VARCHAR(10) DEFAULT NULL NULL
        /

        CREATE TABLE AM_API_PRODUCT_MAPPING (
            API_PRODUCT_MAPPING_ID INTEGER,
            API_ID INTEGER,
            URL_MAPPING_ID INTEGER,
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE,
            FOREIGN KEY (URL_MAPPING_ID) REFERENCES AM_API_URL_MAPPING(URL_MAPPING_ID) ON DELETE CASCADE,
            PRIMARY KEY(API_PRODUCT_MAPPING_ID)
        )
        /

        CREATE SEQUENCE AM_API_PRODUCT_MAPPING_SEQ START WITH 1 INCREMENT BY 1
        /

        CREATE OR REPLACE TRIGGER AM_API_PRODUCT_MAPPING_TRIGGER
        BEFORE INSERT
        ON AM_API_PRODUCT_MAPPING
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT AM_API_PRODUCT_MAPPING_SEQ.nextval INTO :NEW.API_PRODUCT_MAPPING_ID FROM dual;
        END;
        /

        CREATE TABLE AM_REVOKED_JWT (
        UUID VARCHAR(255) NOT NULL,
        SIGNATURE VARCHAR(2048) NOT NULL,
        EXPIRY_TIMESTAMP NUMBER(19) NOT NULL,
        TENANT_ID INTEGER DEFAULT -1,
        TOKEN_TYPE VARCHAR(15) DEFAULT 'DEFAULT',
        TIME_CREATED TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
        PRIMARY KEY (UUID)
        )
        /

        -- UMA tables --
        CREATE TABLE IDN_UMA_RESOURCE (
            ID INTEGER,
            RESOURCE_ID VARCHAR2(255),
            RESOURCE_NAME VARCHAR2(255),
            TIME_CREATED TIMESTAMP NOT NULL,
            RESOURCE_OWNER_NAME VARCHAR2(255),
            CLIENT_ID VARCHAR2(255),
            TENANT_ID INTEGER DEFAULT -1234,
            USER_DOMAIN VARCHAR2(50),
            PRIMARY KEY (ID)
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_RESOURCE_TRIG
        BEFORE INSERT
        ON IDN_UMA_RESOURCE
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_RESOURCE_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CREATE OR REPLACE PROCEDURE add_index_if_not_exists (query IN VARCHAR2)
          IS
        BEGIN
          execute immediate query;
          dbms_output.put_line(query);
        exception WHEN OTHERS THEN
          dbms_output.put_line( 'Skipped ');
        END;
        /
        
        CALL add_index_if_not_exists('CREATE INDEX IDX_RID ON IDN_UMA_RESOURCE (RESOURCE_ID)')
        /
        
        CALL add_index_if_not_exists('CREATE INDEX IDX_USER ON IDN_UMA_RESOURCE (RESOURCE_OWNER_NAME, USER_DOMAIN)')
        /

        CREATE TABLE IDN_UMA_RESOURCE_META_DATA (
            ID INTEGER,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            PROPERTY_KEY VARCHAR2(40),
            PROPERTY_VALUE VARCHAR2(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_META_DATA_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_RESOURCE_METADATA_TRIG
        BEFORE INSERT
        ON IDN_UMA_RESOURCE_META_DATA
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_RESOURCE_META_DATA_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CREATE TABLE IDN_UMA_RESOURCE_SCOPE (
            ID INTEGER,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            SCOPE_NAME VARCHAR2(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_RESOURCE_SCOPE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_RESOURCE_SCOPE_TRIG
        BEFORE INSERT
        ON IDN_UMA_RESOURCE_SCOPE
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_RESOURCE_SCOPE_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CALL add_index_if_not_exists('CREATE INDEX IDX_RS ON IDN_UMA_RESOURCE_SCOPE (SCOPE_NAME)')
        /

        CREATE TABLE IDN_UMA_PERMISSION_TICKET (
            ID INTEGER,
            PT VARCHAR2(255) NOT NULL,
            TIME_CREATED TIMESTAMP NOT NULL,
            EXPIRY_TIME TIMESTAMP NOT NULL,
            TICKET_STATE VARCHAR2(25) DEFAULT 'ACTIVE',
            TENANT_ID INTEGER DEFAULT -1234,
            PRIMARY KEY (ID)
        )
        /

        CREATE SEQUENCE IDN_UMA_PERMISSION_TICKET_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_PERMISSION_TICKET_TRIG
        BEFORE INSERT
        ON IDN_UMA_PERMISSION_TICKET
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_PERMISSION_TICKET_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CALL add_index_if_not_exists('CREATE INDEX IDX_PT ON IDN_UMA_PERMISSION_TICKET (PT)')
        /
        
        DROP PROCEDURE add_index_if_not_exists
        /

        CREATE TABLE IDN_UMA_PT_RESOURCE (
            ID INTEGER,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_ID) REFERENCES IDN_UMA_PERMISSION_TICKET (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_PT_RESOURCE_TRIG
        BEFORE INSERT
        ON IDN_UMA_PT_RESOURCE
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_PT_RESOURCE_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CREATE TABLE IDN_UMA_PT_RESOURCE_SCOPE (
            ID INTEGER,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_SCOPE_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_PT_RESOURCE (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_SCOPE_ID) REFERENCES IDN_UMA_RESOURCE_SCOPE (ID) ON DELETE CASCADE
        )
        /

        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SCOPE_SEQ START WITH 1 INCREMENT BY 1 NOCACHE
        /

        CREATE OR REPLACE TRIGGER IDN_UMA_PT_RESOURCE_SCOPE_TRIG
        BEFORE INSERT
        ON IDN_UMA_PT_RESOURCE_SCOPE
        REFERENCING NEW AS NEW
        FOR EACH ROW
        BEGIN
            SELECT IDN_UMA_PT_RESOURCE_SCOPE_SEQ.nextval INTO :NEW.ID FROM dual;
        END;
        /

        CREATE TABLE AM_API_CATEGORIES (
            UUID VARCHAR2(50),
            NAME VARCHAR2(255) NOT NULL,
            DESCRIPTION VARCHAR2(1024),
            TENANT_ID INTEGER DEFAULT -1,
            UNIQUE (NAME,TENANT_ID),
            PRIMARY KEY (UUID)
        )
        /

        ALTER TABLE AM_SYSTEM_APPS
        ADD TENANT_DOMAIN VARCHAR2(255) DEFAULT 'carbon.super'
        /

        ALTER TABLE AM_SYSTEM_APPS
        DROP UNIQUE (NAME)
        /

        CREATE TABLE AM_USER (
            USER_ID VARCHAR(255) NOT NULL,
            USER_NAME VARCHAR(255) NOT NULL,
            PRIMARY KEY(USER_ID)
        )
        /

        CREATE TABLE AM_SECURITY_AUDIT_UUID_MAPPING (
            API_ID INTEGER NOT NULL,
            AUDIT_UUID VARCHAR(255) NOT NULL,
            PRIMARY KEY (API_ID),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID)
        )
        /

        DELETE FROM IDN_OAUTH2_SCOPE_BINDING WHERE SCOPE_BINDING IS NULL
        /
        COMMIT;
        /           
        ```
        
        ```tab="PostgreSQL"
        DROP TABLE IF EXISTS AM_SYSTEM_APPS;
        CREATE SEQUENCE AM_API_SYSTEM_APPS_SEQUENCE START WITH 1 INCREMENT BY 1;
        CREATE TABLE IF NOT EXISTS AM_SYSTEM_APPS (
            ID INTEGER NOT NULL DEFAULT NEXTVAL('AM_API_SYSTEM_APPS_SEQUENCE'),
            NAME VARCHAR(50) NOT NULL,
            CONSUMER_KEY VARCHAR(512) NOT NULL,
            CONSUMER_SECRET VARCHAR(512) NOT NULL,
            CREATED_TIME TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
            UNIQUE (NAME),
            UNIQUE (CONSUMER_KEY),
            PRIMARY KEY (ID)
        );

        CREATE TABLE IF NOT EXISTS AM_API_CLIENT_CERTIFICATE (
            TENANT_ID INTEGER NOT NULL,
            ALIAS VARCHAR(45) NOT NULL,
            API_ID INTEGER NOT NULL,
            CERTIFICATE BYTEA NOT NULL,
            REMOVED BOOLEAN NOT NULL DEFAULT '0',
            TIER_NAME VARCHAR (512),
            FOREIGN KEY (API_ID) REFERENCES AM_API (API_ID) ON DELETE CASCADE,
            PRIMARY KEY (ALIAS, TENANT_ID, REMOVED)
        );

        ALTER TABLE AM_POLICY_SUBSCRIPTION ADD MONETIZATION_PLAN VARCHAR(25) NULL DEFAULT NULL,
        ADD FIXED_RATE VARCHAR(15) NULL DEFAULT NULL, 
        ADD BILLING_CYCLE VARCHAR(15) NULL DEFAULT NULL, 
        ADD PRICE_PER_REQUEST VARCHAR(15) NULL DEFAULT NULL, 
        ADD CURRENCY VARCHAR(15) NULL DEFAULT NULL;

        CREATE TABLE IF NOT EXISTS AM_MONETIZATION_USAGE (
            ID VARCHAR(100) NOT NULL,
            STATE VARCHAR(50) NOT NULL,
            STATUS VARCHAR(50) NOT NULL,
            STARTED_TIME VARCHAR(50) NOT NULL,
            PUBLISHED_TIME VARCHAR(50) NOT NULL,
            PRIMARY KEY(ID)
        );

        CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

        ALTER TABLE AM_API_COMMENTS
        DROP COLUMN COMMENT_ID,
        ADD COLUMN COMMENT_ID VARCHAR(255) NOT NULL DEFAULT uuid_generate_v1(),
        ADD PRIMARY KEY (COMMENT_ID);

        ALTER TABLE AM_API_RATINGS
        DROP COLUMN RATING_ID,
        ADD COLUMN RATING_ID VARCHAR(255) NOT NULL DEFAULT uuid_generate_v1(),
        ADD PRIMARY KEY (RATING_ID);

        CREATE TABLE IF NOT EXISTS AM_NOTIFICATION_SUBSCRIBER (
            UUID VARCHAR(255),
            CATEGORY VARCHAR(255),
            NOTIFICATION_METHOD VARCHAR(255),
            SUBSCRIBER_ADDRESS VARCHAR(255) NOT NULL,
            PRIMARY KEY(UUID, SUBSCRIBER_ADDRESS)
        );

        ALTER TABLE AM_EXTERNAL_STORES
        ADD LAST_UPDATED_TIME TIMESTAMP DEFAULT CURRENT_TIMESTAMP;

        ALTER TABLE AM_API
        ADD API_TYPE VARCHAR(10) NULL DEFAULT NULL;

        CREATE SEQUENCE AM_API_PRODUCT_MAPPING_SEQUENCE START WITH 1 INCREMENT BY 1;
        CREATE TABLE IF NOT EXISTS AM_API_PRODUCT_MAPPING (
            API_PRODUCT_MAPPING_ID INTEGER DEFAULT nextval('am_api_product_mapping_sequence'),
            API_ID INTEGER,
            URL_MAPPING_ID INTEGER,
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID) ON DELETE CASCADE,
            FOREIGN KEY (URL_MAPPING_ID) REFERENCES AM_API_URL_MAPPING(URL_MAPPING_ID) ON DELETE CASCADE,
            PRIMARY KEY(API_PRODUCT_MAPPING_ID)
        );

        DROP TABLE IF EXISTS AM_REVOKED_JWT;
        CREATE TABLE IF NOT EXISTS AM_REVOKED_JWT (
            UUID VARCHAR(255) NOT NULL,
            SIGNATURE VARCHAR(2048) NOT NULL,
            EXPIRY_TIMESTAMP BIGINT NOT NULL,
            TENANT_ID INTEGER DEFAULT -1,
            TOKEN_TYPE VARCHAR(15) DEFAULT 'DEFAULT',
            TIME_CREATED TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
            PRIMARY KEY (UUID)
        );

        -- UMA tables --
        DROP TABLE IF EXISTS IDN_UMA_RESOURCE;
        DROP SEQUENCE IF EXISTS IDN_UMA_RESOURCE_SEQ;
        CREATE SEQUENCE IDN_UMA_RESOURCE_SEQ;
        CREATE TABLE IDN_UMA_RESOURCE (
            ID INTEGER DEFAULT NEXTVAL('IDN_UMA_RESOURCE_SEQ') NOT NULL,
            RESOURCE_ID VARCHAR(255),
            RESOURCE_NAME VARCHAR(255),
            TIME_CREATED TIMESTAMP NOT NULL,
            RESOURCE_OWNER_NAME VARCHAR(255),
            CLIENT_ID VARCHAR(255),
            TENANT_ID INTEGER DEFAULT -1234,
            USER_DOMAIN VARCHAR(50),
            PRIMARY KEY (ID)
        );

        CREATE INDEX IF NOT EXISTS IDX_RID ON IDN_UMA_RESOURCE (RESOURCE_ID);

        CREATE INDEX IF NOT EXISTS IDX_USER ON IDN_UMA_RESOURCE (RESOURCE_OWNER_NAME, USER_DOMAIN);

        DROP TABLE IF EXISTS IDN_UMA_RESOURCE_META_DATA;
        DROP SEQUENCE IF EXISTS IDN_UMA_RESOURCE_META_DATA_SEQ;
        CREATE SEQUENCE IDN_UMA_RESOURCE_META_DATA_SEQ;
        CREATE TABLE IDN_UMA_RESOURCE_META_DATA (
            ID INTEGER DEFAULT NEXTVAL ('IDN_UMA_RESOURCE_META_DATA_SEQ') NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            PROPERTY_KEY VARCHAR(40),
            PROPERTY_VALUE VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        DROP TABLE IF EXISTS IDN_UMA_RESOURCE_SCOPE;
        DROP SEQUENCE IF EXISTS IDN_UMA_RESOURCE_SCOPE_SEQ;
        CREATE SEQUENCE IDN_UMA_RESOURCE_SCOPE_SEQ;
        CREATE TABLE IDN_UMA_RESOURCE_SCOPE (
            ID INTEGER DEFAULT NEXTVAL ('IDN_UMA_RESOURCE_SCOPE_SEQ') NOT NULL,
            RESOURCE_IDENTITY INTEGER NOT NULL,
            SCOPE_NAME VARCHAR(255),
            PRIMARY KEY (ID),
            FOREIGN KEY (RESOURCE_IDENTITY) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        CREATE INDEX IF NOT EXISTS IDX_RS ON IDN_UMA_RESOURCE_SCOPE (SCOPE_NAME);

        DROP TABLE IF EXISTS IDN_UMA_PERMISSION_TICKET;
        DROP SEQUENCE IF EXISTS IDN_UMA_PERMISSION_TICKET_SEQ;
        CREATE SEQUENCE IDN_UMA_PERMISSION_TICKET_SEQ;
        CREATE TABLE IDN_UMA_PERMISSION_TICKET (
            ID INTEGER DEFAULT NEXTVAL('IDN_UMA_PERMISSION_TICKET_SEQ') NOT NULL,
            PT VARCHAR(255) NOT NULL,
            TIME_CREATED TIMESTAMP NOT NULL,
            EXPIRY_TIME TIMESTAMP NOT NULL,
            TICKET_STATE VARCHAR(25) DEFAULT 'ACTIVE',
            TENANT_ID INTEGER DEFAULT -1234,
            PRIMARY KEY (ID)
        );

        CREATE INDEX IF NOT EXISTS IDX_PT ON IDN_UMA_PERMISSION_TICKET (PT);

        DROP TABLE IF EXISTS IDN_UMA_PT_RESOURCE;
        DROP SEQUENCE IF EXISTS IDN_UMA_PT_RESOURCE_SEQ;
        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SEQ;
        CREATE TABLE IDN_UMA_PT_RESOURCE (
            ID INTEGER DEFAULT NEXTVAL ('IDN_UMA_PT_RESOURCE_SEQ') NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_ID) REFERENCES IDN_UMA_PERMISSION_TICKET (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_RESOURCE (ID) ON DELETE CASCADE
        );

        DROP TABLE IF EXISTS IDN_UMA_PT_RESOURCE_SCOPE;
        DROP SEQUENCE IF EXISTS IDN_UMA_PT_RESOURCE_SCOPE_SEQ;
        CREATE SEQUENCE IDN_UMA_PT_RESOURCE_SCOPE_SEQ;
        CREATE TABLE IF NOT EXISTS IDN_UMA_PT_RESOURCE_SCOPE (
            ID INTEGER DEFAULT NEXTVAL ('IDN_UMA_PT_RESOURCE_SCOPE_SEQ') NOT NULL,
            PT_RESOURCE_ID INTEGER NOT NULL,
            PT_SCOPE_ID INTEGER NOT NULL,
            PRIMARY KEY (ID),
            FOREIGN KEY (PT_RESOURCE_ID) REFERENCES IDN_UMA_PT_RESOURCE (ID) ON DELETE CASCADE,
            FOREIGN KEY (PT_SCOPE_ID) REFERENCES IDN_UMA_RESOURCE_SCOPE (ID) ON DELETE CASCADE
        );

        DROP TABLE IF EXISTS AM_API_CATEGORIES;
        CREATE TABLE IF NOT EXISTS AM_API_CATEGORIES (
            UUID VARCHAR(50),
            NAME VARCHAR(255),
            DESCRIPTION VARCHAR(1024),
            TENANT_ID INTEGER DEFAULT -1,
            UNIQUE (NAME,TENANT_ID),
            PRIMARY KEY (UUID)
        );

        ALTER TABLE AM_SYSTEM_APPS
        ADD TENANT_DOMAIN VARCHAR(255) DEFAULT 'carbon.super';

        ALTER TABLE AM_SYSTEM_APPS
        DROP CONSTRAINT AM_SYSTEM_APPS_NAME_KEY;

        DROP TABLE IF EXISTS AM_USER;
        CREATE TABLE IF NOT EXISTS AM_USER (
            USER_ID VARCHAR(255) NOT NULL,
            USER_NAME VARCHAR(255) NOT NULL,
            PRIMARY KEY(USER_ID)
        );

        DROP TABLE IF EXISTS AM_SECURITY_AUDIT_UUID_MAPPING;
        CREATE TABLE IF NOT EXISTS AM_SECURITY_AUDIT_UUID_MAPPING (
            API_ID INTEGER NOT NULL,
            AUDIT_UUID VARCHAR(255) NOT NULL,
            PRIMARY KEY (API_ID),
            FOREIGN KEY (API_ID) REFERENCES AM_API(API_ID)
        );
        
        DELETE FROM IDN_OAUTH2_SCOPE_BINDING WHERE SCOPE_BINDING IS NULL OR SCOPE_BINDING = '';
        ```

4.  Copy the keystores (i.e., `client-truststore.jks`, `wso2cabon.jks` and any other custom JKS) used in the previous version and replace the existing keystores in the `<API-M_3.1.0_HOME>/repository/resources/security` directory.

    !!! note "From API Manager 3.0.0 onwards, keystores could be configured as follows"
    
        - `keystore.primary` - The default keystore. If only the primary keystore is configured, the primary keystore will be used for internal and SSL related encryption and decryption tasks. 
        - `keystore.internal` - The keystore used for internal encryption/decryption.
        - `keystore.tls` - The keystore used for SSL encryption.

    !!! Attention
        In API Manager 3.1.0, it is required to use a certificate with the RSA key size greater than 2048. If you have used a certificate that has a weak RSA key (key size less than 2048) in the previous version, you need to add the following configuration to the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file to configure internal and primary keystores. You should point the internal keystore to the keystore copied from API Manager 2.6.0 and primary keystore can be pointed to a keystore with a certificate, which has a strong RSA key. 

        ``` java
        [keystore.primary]
        file_name = "primary.jks"
        type = "JKS"
        password = "wso2carbon"
        alias = "wso2carbon"
        key_password = "wso2carbon"

        [keystore.internal]
        file_name = "internal.jks"
        type = "JKS"
        password = "wso2carbon"
        alias = "wso2carbon"
        key_password = "wso2carbon"
        ```

    !!! note "If you have enabled Secure Vault"
        If you have enabled secure vault in the previous API-M version, you need to add the property values again according to the new config model and run the script as below. For more information, see [Encrypting Passwords in Configuration files]({{base_path}}/install-and-setup/setup/security/logins-and-passwords/working-with-encrypted-passwords).

        ```tab="Linux"
        ./ciphertool.sh -Dconfigure
        ```

        ```tab="Windows"
        ./ciphertool.bat -Dconfigure
        ```
    - If the truststore used in the previous setup was changed (e.g., the name of the truststore file or the password was changed), copy the truststore to the `<API-M_3.1.0_HOME>/repository/resources/security` directory and update the credentials in the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file as follows:

       ```toml
       [truststore]
       type= "JKS"
       file_name = "modified-client-truststore.jks"
       password= "modified_password"
       ```

    - In order to work with the [API Security Audit Feature]({{base_path}}/learn/api-security/configuring-api-security-audit/) you need to have the public certificate of the [42crunch](https://42crunch.com/) in the client-truststore. Follow the guidelines given in [Importing Certificates to the Truststore]({{base_path}}/install-and-setup/setup/security/configuring-keystores/keystore-basics/creating-new-keystores/#step-3-importing-certificates-to-the-truststore).
    
5.  Upgrade the Identity component in WSO2 API Manager from version 5.7.0 to 5.10.0.

    !!! note

        If you are using WSO2 Identity Server (WSO2 IS) as a Key Manager, you have to follow the instructions in [Upgrading WSO2 IS as the Key Manager to 5.10.0]({{base_path}}/install-and-setup/upgrading-wso2-is-as-key-manager/upgrading-from-is-km-570-to-5100) instead of the instructions below.

        But, if you are not using WSO2 IS as a Key Manager, you have to follow the instructions below in order to upgrade the identity components which have been shared with WSO2 API-M.

    ??? note "If you are using DB2"
        Move indexes to the the TS32K Tablespace. The index tablespace in the `IDN_OAUTH2_ACCESS_TOKEN` and `IDN_OAUTH2_AUTHORIZATION_CODE` tables need to be moved to the existing TS32K tablespace in order to support the newly added table indexes.

        SQLADM or DBADM authority is required in order to invoke the `ADMIN_MOVE_TABLE` stored procedure. You must also have the appropriate object creation authorities, including authorities to issue the SELECT statement on the source table and to issue the INSERT statement on the target table.    

        ??? info "Click here to see the stored procedure" 
            ``` java
            CREATE BUFFERPOOL BP32K IMMEDIATE SIZE 250 AUTOMATIC PAGESIZE 32K
            /
            CREATE LARGE TABLESPACE TS32K PAGESIZE 32K MANAGED by AUTOMATIC STORAGE BUFFERPOOL BP32K
            /
            
            CALL SYSPROC.ADMIN_MOVE_TABLE(
            <TABLE_SCHEMA_OF_IDN_OAUTH2_ACCESS_TOKEN_TABLE>,
            'IDN_OAUTH2_ACCESS_TOKEN',
            (SELECT TBSPACE FROM SYSCAT.TABLES where TABNAME = 'IDN_OAUTH2_ACCESS_TOKEN' AND TABSCHEMA = <TABLE_SCHEMA_OF_IDN_OAUTH2_ACCESS_TOKEN_TABLE>),
            'TS32K',
            (SELECT TBSPACE FROM SYSCAT.TABLES where TABNAME = 'IDN_OAUTH2_ACCESS_TOKEN' AND TABSCHEMA = <TABLE_SCHEMA_OF_IDN_OAUTH2_ACCESS_TOKEN_TABLE>),
            '',
            '',
            '',
            '',
            '',
            'MOVE');

            CALL SYSPROC.ADMIN_MOVE_TABLE(
            <TABLE_SCHEMA_OF_IDN_OAUTH2_AUTHORIZATION_CODE_TABLE>,
            'IDN_OAUTH2_AUTHORIZATION_CODE',
            (SELECT TBSPACE FROM SYSCAT.TABLES where TABNAME = 'IDN_OAUTH2_AUTHORIZATION_CODE' AND TABSCHEMA = <TABLE_SCHEMA_OF_IDN_OAUTH2_AUTHORIZATION_CODE_TABLE>),
            'TS32K',
            (SELECT TBSPACE FROM SYSCAT.TABLES where TABNAME = 'IDN_OAUTH2_AUTHORIZATION_CODE' AND TABSCHEMA = <TABLE_SCHEMA_OF_IDN_OAUTH2_AUTHORIZATION_CODE_TABLE>),
            '',
            '',
            '',
            '',
            '',
            'MOVE');

            Where,

            <TABLE_SCHEMA_OF_IDN_OAUTH2_ACCESS_TOKEN_TABLE> and <TABLE_SCHEMA_OF_IDN_OAUTH2_AUTHORIZATION_CODE_TABLE> : Replace these schema’s with each respective schema for the table.
            ```

        If you get an error due to missing `SYSTOOLSPACE` or `SYSTOOLSTMPSPACE` tablespaces, create those tablespaces manually using the following script prior to executing the stored procedure given above. For more information, see [SYSTOOLSPACE and SYSTOOLSTMPSPACE table
        spaces](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.5.0/com.ibm.db2.luw.admin.gui.doc/doc/c0023713.html) in the official IBM documentation.

        ``` java
        CREATE TABLESPACE SYSTOOLSPACE IN IBMCATGROUP
        MANAGED BY AUTOMATIC STORAGE USING STOGROUP IBMSTOGROUP
        EXTENTSIZE 4;
        
        CREATE USER TEMPORARY TABLESPACE SYSTOOLSTMPSPACE IN IBMCATGROUP
        MANAGED BY AUTOMATIC STORAGE USING STOGROUP IBMSTOGROUP
        EXTENTSIZE 4;
        ```

    1.  Download the identity component migration resources and unzip it in a local directory.

         Navigate to the [latest release tag](https://github.com/wso2-extensions/identity-migration-resources/releases/latest) and download the `wso2is-migration-x.x.x.zip` under **Assets**. Let's refer to this directory that you downloaded and extracted as `<IS_MIGRATION_TOOL_HOME>`. 

    2.  Copy the `migration-resources` folder from the extracted folder to the `<API-M_3.1.0_HOME>` directory.

    3.  Open the `migration-config.yaml` file in the `migration-resources` directory and make sure that the `currentVersion` element is set to 5.7.0, as shown below.

        ``` java
        migrationEnable: "true"
        currentVersion: "5.7.0"
        migrateVersion: "5.10.0"
        ```

        !!! note
            Make sure you have enabled migration by setting the `migrationEnable` element to `true` as shown above. You have to remove the following 2 steps from the `migration-config.yaml` file which is included under version: "5.10.0"
                ```
                -
                    name: "MigrationValidator"
                    order: 2
                -
                    name: "SchemaMigrator"
                    order: 5
                    parameters:
                    location: "step2"
                    schema: "identity"
                ```

    4.  Copy the `org.wso2.carbon.is.migration-x.x.x.jar` from the `<IS_MIGRATION_TOOL_HOME>/dropins` directory to the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

    5. Update the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file as follows to point to the previous user store.
    
        ```
        [user_store]
        type = "database"
        ```
    
    6.  Start WSO2 API Manager 3.1.0 as follows to carry out the complete Identity component migration.
        
        !!! note
            If you are migrating your user stores to the new user store managers with the unique ID capabilities, follow the guidelines given in the [Migrating User Store Managers](https://is.docs.wso2.com/en/latest/setup/migrating-userstore-managers/) before moving to the next step.
                    
        ```tab="Linux / Mac OS"
        sh wso2server.sh -Dmigrate -Dcomponent=identity
        ```

        ```tab="Windows"
        wso2server.bat -Dmigrate -Dcomponent=identity
        ```

        !!! note
            Note that depending on the number of records in the identity tables, this identity component migration will take a considerable amount of time to finish. Do not stop the server during the migration process and wait until the migration process finishes completely and the server starts.
        
        !!! note
            If you want to use the latest user store, update the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` as follows after the identity migration.

            ```
            [user_store]
            type = "database_unique_id"
            ``` 

        !!! warning "Troubleshooting"
            When running the above step if you encounter the following error message, follow the steps in this section. Note that this error could occur only if the identity tables contain a huge volume of data.

            The sample exception stack trace is given below.

            ```
            ERROR {org.wso2.carbon.registry.core.dataaccess.TransactionManager} -  Failed to start new registry transaction. {org.wso2.carbon.registry.core.dataaccess.TransactionManager} org.apache.tomcat.jdbc.pool.PoolExhaustedException: [pool-30-thread-11] Timeout: Pool empty. Unable to fetch a connection in 60 seconds, none available[size:50; busy:50; idle:0; lastwait:60000
            ```

            1. Add the following property in `<API-M_HOME>/repository/conf/deployment.toml` to a higher value (e.g., 10)
                ```
                [indexing]
                frequency= 10
                ```

            2. Re-run the command above.

            **Make sure to revert the change done in Step (i.), after the migration is complete.**

    7.  After you have successfully completed the migration, stop the server and remove the following files and folders.

        -   Remove the `org.wso2.carbon.is.migration-x.x.x.jar` file, which is in the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

        -   Remove the `migration-resources` directory, which is in the `<API-M_3.1.0_HOME>` directory.

        -   If you ran WSO2 API-M as a Windows Service when doing the identity component migration, then you need to remove the following parameters in the command line arguments section (CMD_LINE_ARGS) of the `wso2server.bat` file.

            ```
            -Dmigrate -Dcomponent=identity
            ```

6.  Migrate the API Manager artifacts.

    You have to run the following migration client to update the registry artifacts.

    1. Download and extract the [migration-resources.zip]({{base_path}}/assets/attachments/install-and-setup/migration-resources.zip). Copy the extracted `migration-resources` to the `<API-M_3.1.0_HOME>` folder.

    2. Download and copy the [API Manager Migration Client]({{base_path}}/assets/attachments/install-and-setup/org.wso2.carbon.apimgt.migrate.client-3.1.0-6.jar) to the `<API-M_3.1.0_HOME>/repository/components/dropins` folder.

    3.  Start the API-M server as follows.

        ``` tab="Linux / Mac OS"
        sh wso2server.sh -DmigrateFromVersion=2.6.0
        ```

        ``` tab="Windows"
        wso2server.bat -DmigrateFromVersion=2.6.0
        ```

    4. Shutdown the API-M server.
    
       -   Remove the `org.wso2.carbon.apimgt.migrate.client-3.1.0-6.jar` file, which is in the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

       -   Remove the `migration-resources` directory, which is in the `<API-M_3.1.0_HOME>` directory.

7.  Re-index the artifacts in the registry.
    1.  Run the [reg-index.sql]({{base_path}}/assets/attachments/install-and-setup/reg-index.sql) script against the `SHARED_DB` database.

        !!! note
            Depending on the number of records in the `REG_LOG` table this script will take a considerable amount of time to finish. Do not stop the execution of the script until it is completed.

    2.  Add the [tenantloader-1.0.jar]({{base_path}}/assets/attachments/install-and-setup/tenantloader-1.0.jar) to `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

        !!! attention
            If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Store and Publisher** nodes.

        !!! note
            You need to do this step if you have **multiple tenants** only.

    3.  Add the following configuration in the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file.
        
        ```
        [indexing]
        re_indexing= 1
        ```

        !!! info 
             If you use a clustered/distributed API Manager setup, do the above change in the `deployment.toml` of the Publisher and the Developer Portal nodes.
             
    4.  If the `<API-M_3.1.0_HOME>/solr` directory exists, take a backup and thereafter delete it.

    5.  Start the WSO2 API-M server.

    6.  Stop the WSO2 API-M server and remove the `tenantloader-1.0.jar` from the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

### Step 3 - Optionally, migrate the configurations for WSO2 API-M Analytics

!!! warning
    This step is **only required** if you have WSO2 API-M-Analytics configured in your current deployment.

Follow the steps below to migrate APIM Analytics 2.6.0 to APIM Analytics 3.1.0

#### Step 3.1 - Migrate the Analytics Database

Upgrade the WSO2 API Manager Analytics database from version 2.6.0 to version 3.1.0 by executing the relevant database script, from the scripts that are provided below, on the `APIM_ANALYTICS_DB` database.

??? info "DB Scripts"
    ```tab="DB2"
    ALTER TABLE APILASTACCESSSUMMARY DROP PRIMARY KEY;
    ALTER TABLE APILASTACCESSSUMMARY ALTER COLUMN APIVERSION VARCHAR(254) NOT NULL;
    ALTER TABLE APILASTACCESSSUMMARY ADD PRIMARY KEY (APINAME,APIVERSION,APICREATOR,APICREATORTENANTDOMAIN);
    ```

    ```tab="MSSQL"
    DECLARE @con_com as VARCHAR(8000);
    SET @con_com = (SELECT name from sys.objects where parent_object_id=object_id('APILASTACCESSSUMMARY') AND type='PK');
    EXEC('ALTER TABLE APILASTACCESSSUMMARY DROP CONSTRAINT ' + @con_com);
    ALTER TABLE APILASTACCESSSUMMARY ALTER COLUMN APIVERSION VARCHAR(254) NOT NULL;
    ALTER TABLE APILASTACCESSSUMMARYADD PRIMARY KEY (APINAME,APICREATOR,APIVERSION,APICREATORTENANTDOMAIN);
    ```

    ```tab="MySQL"
    ALTER TABLE APILASTACCESSSUMMARY DROP PRIMARY KEY;
    ALTER TABLE APILASTACCESSSUMMARY ADD PRIMARY KEY (APINAME,APICREATOR,APIVERSION,APICREATORTENANTDOMAIN);
    ```
    
    ```tab="Oracle"
    ALTER TABLE APILASTACCESSSUMMARY DROP PRIMARY KEY;
    ALTER TABLE APILASTACCESSSUMMARY ADD PRIMARY KEY (APINAME,APICREATOR,APIVERSION,APICREATORTENANTDOMAIN);
    COMMIT;
    ```
        
    ```tab="PostgreSQL"
    ALTER TABLE APILASTACCESSSUMMARY DROP CONSTRAINT APILASTACCESSSUMMARY_pkey;
    ALTER TABLE APILASTACCESSSUMMARY ADD PRIMARY KEY (APINAME,APICREATOR,APIVERSION,APICREATORTENANTDOMAIN);
    ```
!!! note
    Type and name of a column of few tables were changed through WUM in analytics version 2.6. It is important to 
    add the above change into your database prior to migration. Therefore, execute the following queries that check whether 
    the above change is already available in your DB. If the above mentioned change is not available, it will add the relevant change. Ensure to replace `"<Enter Analytics DB name here>"` with the correct DB name in the scripts.
    
    ```tab="MSSQL"
    CREATE PROCEDURE dbo.alter_geolocation_table_if_coloumn_not_exist
    AS
    BEGIN
    IF NOT EXISTS(
    	SELECT name  FROM SYS.COLUMNS
        	WHERE OBJECT_ID = OBJECT_ID('geolocationagg_seconds') AND NAME = 'agg_count')
    	BEGIN
    		ALTER TABLE GeoLocationAgg_SECONDS ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_SECONDS.totalCount' , 'AGG_COUNT', 'COLUMN';
    		ALTER TABLE GeoLocationAgg_MINUTES ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_MINUTES.totalCount' , 'AGG_COUNT', 'COLUMN';
    		ALTER TABLE GeoLocationAgg_HOURS ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_HOURS.totalCount' , 'AGG_COUNT', 'COLUMN';
    		ALTER TABLE GeoLocationAgg_DAYS ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_DAYS.totalCount' , 'AGG_COUNT', 'COLUMN';
    		ALTER TABLE GeoLocationAgg_MONTHS ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_MONTHS.totalCount' , 'AGG_COUNT', 'COLUMN';
    		ALTER TABLE GeoLocationAgg_YEARS ALTER COLUMN totalCount BIGINT;
    		EXEC sp_RENAME 'GeoLocationAgg_YEARS.totalCount' , 'AGG_COUNT', 'COLUMN';
    	END 	
    END
    GO
    
    USE <Enter Analytics DB name here>;
    EXEC dbo.alter_geolocation_table_if_coloumn_not_exist;
    DROP PROCEDURE dbo.alter_geolocation_table_if_coloumn_not_exist;
    ```
    
    ```tab="MySQL"
    DROP PROCEDURE IF EXISTS alter_geolocation_table_if_coloumn_not_exist;
    
    DELIMITER $$
    
    CREATE DEFINER=CURRENT_USER PROCEDURE alter_geolocation_table_if_coloumn_not_exist (IN dbName varchar(50)) 
    BEGIN
    	DECLARE colName TEXT;
    	SELECT column_name INTO colName
    	FROM information_schema.columns WHERE table_schema = dbName AND table_name = 'GeoLocationAgg_SECONDS'
    	AND column_name = 'AGG_COUNT'; 
    	IF colName is null THEN 
    		ALTER TABLE GeoLocationAgg_SECONDS CHANGE totalCount AGG_COUNT bigint(20);
    		ALTER TABLE GeoLocationAgg_MINUTES CHANGE totalCount AGG_COUNT bigint(20);
    		ALTER TABLE GeoLocationAgg_HOURS CHANGE totalCount AGG_COUNT bigint(20);
    		ALTER TABLE GeoLocationAgg_DAYS CHANGE totalCount AGG_COUNT bigint(20);
    		ALTER TABLE GeoLocationAgg_MONTHS CHANGE totalCount AGG_COUNT bigint(20);
    		ALTER TABLE GeoLocationAgg_YEARS CHANGE totalCount AGG_COUNT bigint(20);
    	END IF; 
    END$$
    
    DELIMITER ;
    CALL alter_geolocation_table_if_coloumn_not_exist('<Enter Analytics DB name here>');
    ```
    
    ```tab="Oracle"
    DECLARE
      column_exists number := 0;  
    BEGIN
      Select count(*) into column_exists
      from user_tab_cols
      where upper(column_name) = 'AGG_COUNT'
      and upper(table_name) = 'GEOLOCATIONAGG_SECONDS';
          
      IF (column_exists = 0) then
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_SECONDS modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_SECONDS RENAME COLUMN totalCount to AGG_COUNT';           
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_MINUTES modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_MINUTES RENAME COLUMN totalCount to AGG_COUNT';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_HOURS modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_HOURS RENAME COLUMN totalCount to AGG_COUNT';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_DAYS modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_DAYS RENAME COLUMN totalCount to AGG_COUNT';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_MONTHS modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_MONTHS RENAME COLUMN totalCount to AGG_COUNT';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_YEARS modify (totalCount INTEGER DEFAULT 0)';
           EXECUTE IMMEDIATE 'ALTER TABLE GeoLocationAgg_YEARS RENAME COLUMN totalCount to AGG_COUNT';   
      END IF;
    END;
    /
    COMMIT;
    /
    ```
    
    ```tab="PostgreSQL"
    CREATE OR REPLACE FUNCTION alter_geolocation_table_if_coloumn_not_exist(IN dbName varchar(50))
      returns void AS $$
      declare
       tableName VARCHAR(50) := 'geolocationagg_seconds';
       tableColumn VARCHAR(50) := 'agg_count'; 
       colName VARCHAR(50);
      begin
    	  select column_name into colName FROM information_schema.columns where table_catalog = dbName and 
    	  table_name = tableName and column_name = tableColumn;
    	  if colName is null then
    	  	ALTER TABLE GeoLocationAgg_SECONDS ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_SECONDS rename totalCount to AGG_COUNT;
    		ALTER TABLE GeoLocationAgg_MINUTES ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_MINUTES rename totalCount to AGG_COUNT;
    		ALTER TABLE GeoLocationAgg_HOURS ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_HOURS rename totalCount to AGG_COUNT;
    		ALTER TABLE GeoLocationAgg_DAYS ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_DAYS rename totalCount to AGG_COUNT;
    		ALTER TABLE GeoLocationAgg_MONTHS ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_MONTHS rename totalCount to AGG_COUNT;
    		ALTER TABLE GeoLocationAgg_YEARS ALTER COLUMN totalCount TYPE INTEGER;
    		ALTER TABLE GeoLocationAgg_YEARS rename totalCount to AGG_COUNT;
    	 end if;
      END; $$
      LANGUAGE plpgsql;
    select alter_geolocation_table_if_coloumn_not_exist(<Enter Analytics DB name here>);
    ```
    
    ```tab="db2"
    CREATE OR REPLACE PROCEDURE alter_geolocation_table_if_coloumn_not_exist ()
         MODIFIES SQL DATA
         LANGUAGE SQL
    BEGIN
        IF (NOT EXISTS(SELECT 1 FROM SYSCAT.COLUMNS WHERE TABNAME = 'GEOLOCATIONAGG_SECONDS' AND COLNAME = 'AGG_COUNT'))
        THEN
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_SECONDS ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_SECONDS RENAME COLUMN totalCount TO AGG_COUNT';   
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_MINUTES ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_MINUTES RENAME COLUMN totalCount TO AGG_COUNT';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_HOURS ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_HOURS RENAME COLUMN totalCount TO AGG_COUNT';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_DAYS ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_DAYS RENAME COLUMN totalCount TO AGG_COUNT';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_MONTHS ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_MONTHS RENAME COLUMN totalCount TO AGG_COUNT';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_YEARS ALTER COLUMN TOTALCOUNT SET DATA TYPE INTEGER';
            EXECUTE IMMEDIATE 'ALTER TABLE GEOLOCATIONAGG_YEARS RENAME COLUMN totalCount TO AGG_COUNT';
        END IF;
    END
    /
    
    CALL alter_geolocation_table_if_coloumn_not_exist()
    /

    ```    
    
#### Step 3.2 - Configure WSO2 API-M Analytics 3.1.0

!!! note
    -   In API-M 2.6.0, when working with API-M Analytics, only the worker profile has been used by default and the dashboard profile is used only when there are custom dashboards.
    -   Now with API-M 3.1.0, both the worker and dashboard profiles are being used. The default Store and Publisher dashboards are now being moved to the Analytics dashboard server side and they have been removed from the API-M side.
    -   The same set of DBs will be used on the Analytics side. In addition, you need to share the `WSO2AM_DB` with the dashboard server node.

!!! info
    Sometimes due to case insensitivity of primary keys in aggregation tables, primary key violation errors are thrown when you try to insert a new record with the same value as an existing record. To overcome this, you need to add encoding and collation to database when the Analytics DB is created (i.e., before the tables are created). For more information on collation, see [MySQL](https://dev.mysql.com/doc/refman/5.7/en/charset-collation-names.html) or [MS SQL](https://docs.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-ver15) based on the database that you are using. Sample commands are provided below.

    !!! example

        ```sql tab="MySQL"
        ALTER DATABASE <DB-NAME> COLLATE latin1_general_cs ;
        ```
        
        ```sql tab="MS SQL"
        ALTER DATABASE <DB-NAME> COLLATE SQL_Latin1_General_CP1_CS_AS ;
        ```

Follow the instructions below to configure WSO2 API Manager Analytics for the WSO2 API-M Analytics migration in order to migrate the statistics related data.

1.  Download [WUM updated](https://docs.wso2.com/display/updates/Getting+Started) pack for [WSO2 API Manager Analytics 3.1.0](http://wso2.com/api-management/).

    !!! note
        It is **mandatory** to use a WUM updated WSO2 API Manager Analytics 3.1.0 pack when migrating the configurations for WSO2 API-M Analytics.

2.  Configure the following 2 datasources in the `<API-M_ANALYTICS_3.1.0_HOME>/conf/dashboard/deployment.yaml` file by pointing to the **old** `WSO2AM_DB` and `APIM_ANALYTICS_DB`.

    ``` java
    #Data source for APIM Analytics
    - name: APIM_ANALYTICS_DB
        description: Datasource used for APIM Analytics
        jndiConfig:
        name: jdbc/APIM_ANALYTICS_DB
        definition:
        type: RDBMS
        configuration:
            jdbcUrl: 'jdbc:mysql://localhost:3306/analytics_db'
            username: root
            password: root
            driverClassName: com.mysql.jdbc.Driver
            minIdle: 5
            maxPoolSize: 50
            idleTimeout: 60000
            connectionTestQuery: SELECT 1
            validationTimeout: 30000
            isAutoCommit: false

    #Main datasource used in API Manager
    - name: AM_DB
        description: Main datasource used by API Manager
        jndiConfig:
        name: jdbc/AM_DB
        definition:
        type: RDBMS
        configuration:
            jdbcUrl: "jdbc:mysql://localhost:3306/am_db"
            username: root
            password: root
            driverClassName: com.mysql.jdbc.Driver
            maxPoolSize: 10
            idleTimeout: 60000
            connectionTestQuery: SELECT 1
            validationTimeout: 30000
            isAutoCommit: false
    ```

3.  Configure the following datasource in the `<API-M_ANALYTICS_3.1.0_HOME>/conf/worker/deployment.yaml` file by pointing to the **old** `APIM_ANALYTICS_DB`.

    ``` java
    #Data source for APIM Analytics
    - name: APIM_ANALYTICS_DB
      description: "The datasource used for APIM statistics aggregated data."
      jndiConfig:
        name: jdbc/APIM_ANALYTICS_DB
      definition:
        type: RDBMS
        configuration:
          jdbcUrl: 'jdbc:mysql://localhost:3306/analytics_db'
          username: root
          password: root
          driverClassName: com.mysql.jdbc.Driver
          minIdle: 5
          maxPoolSize: 50
          idleTimeout: 60000
          connectionTestQuery: SELECT 1
          validationTimeout: 30000
          isAutoCommit: false
    ```

4.  Copy the relevant JDBC driver OSGI bundle to the `<APIM_ANALYTICS_3.1.0_HOME>/lib` folder.

    !!! info "To convert the JAR files to OSGi bundles, follow the steps given below."
        1. Download the non-OSGi JAR for the required third party product, and save it in a preferred directory in your machine.
        2. Go to the `<API-M_ANALYTICS_HOME>/bin` directory. Run the command given below, to generate the converted file in the `<API-M_ANALYTICS_HOME>/lib` directory.

        ```
        ./jartobundle.sh <PATH_TO_NON-OSGi_JAR> ../lib
        ```

5.  Copy the keystores (i.e., `client-truststore.jks`, `wso2cabon.jks` and any other custom JKS) used in the previous version from `<OLD_API-M_ANALYTICS_HOME>/repository/resources/security` and replace the existing keystores in the `<NEW_API-M_ANALYTICS_HOME>/resources/security` directory.

6.  Start the Worker and Dashboard profiles as below by navigating to `<API-M_ANALYTICS_3.1.0_HOME>/bin` location.
    
    ```tab="Worker"
    sh worker.sh
    ```

    ```tab="Dashboard"
    sh dashboard.sh
    ```

!!! note
    If you have developed any custom dashboards in API-M 2.6.0 Analytics using Stream Processor, you will be able to use the same in API-M Anaytics 3.1.0 as well. If you require any guidance regarding this, you can contact [WSO2 Support](https://support.wso2.com/jira/secure/Dashboard.jspa).

#### Step 3.3 - Configure WSO2 API-M 3.1.0 for Analytics

Enable analytics in WSO2 API-M by setting the following configuration to `true` in the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file.

    ``` java
    [apim.analytics]
    enable = true
    ```

### Step 4 - Restart the WSO2 API-M 3.1.0 server

Restart the WSO2 API-M server.

```tab="Linux / Mac OS"
sh wso2server.sh
```

```tab="Windows"
wso2server.bat
```

!!! note "If you have enabled Analytics"
    After starting the WSO2 API-M server and the WSO2 API-M Analytics 3.1.0 server from worker and dashboard profiles, the dashboards can be accessed via `https://<dashboard-server-host-name>:9643/analytics-dashboard` link.

    !!! warning
        Make sure you have started the API-M server node before accessing the Dashboard profile as the authentication happens via the API-M's authentication admin service.

    If you are using an external IDP through a federated authenticator, you have to edit the "sp_analytics_dashboard" service provider configuration and select the "Use tenant domain at local subject identifier" under the **Local & Outbound Authentication** configuration section.
        
    [![select local subject identifier]({{base_path}}/assets/img/setup-and-install/migration-analytics-fedarated-authentication.png)]({{base_path}}/assets/img/setup-and-install/migration-analytics-fedarated-authentication.png)

This concludes the upgrade process.

!!! tip
    The migration client that you use in this guide automatically migrates your tenants, workflows, external user stores, etc. to the upgraded environment. Therefore, there is no need to migrate them manually.

!!! note
    If you are using a migrated API and you need to consume it via an application that supports JWT authentication (default type in API-M 3.1.0), you need to re-publish the API. Without republishing the API, the JWT authentication does not work as it looks for a local entry that will get populated while publishing.

    You can consume the migrated API via an OAuth2 application without an issue.
