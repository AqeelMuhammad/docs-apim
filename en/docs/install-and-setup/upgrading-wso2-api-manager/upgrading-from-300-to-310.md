# Upgrading API Manager from 3.0.0 to 3.1.0

The following information describes how to upgrade your API Manager server **from APIM 3.0.0 to 3.1.0**.

!!! note
    Before you follow this section, see [Upgrading Process]({{base_path}}/install-and-setup/upgrading-wso2-api-manager/upgrading-process) for more information.

!!! attention "Before you Begin"
    This release is a WUM-only release. This means that there are no manual patches. Any further fixes or latest updates for this release can be updated through the WSO2 Update Manager (WUM).

    -   **If you are upgrading to this version, in order to use this version in your production environment** , use the WSO2 Update Manager and get the latest available updates for WSO2 API Manager 3.1.0. For more information on how to do this, see [Updating API Manager]({{base_path}}/administer/updating-wso2-api-manager/#wso2-update-manager-wum).

!!! note "If you are using PostgreSQL"
    The DB user needs to have superuser role to run the migration client and the relevant scripts
    ```
    ALTER USER <user> WITH SUPERUSER;
    ```
!!! note "If you are using Oracle"
    Please commit the changes after running the scripts given below
    
Follow the instructions below to upgrade your WSO2 API Manager server **from WSO2 API-M 3.0.0 to 3.1.0**.

### Preparing for Migration
#### Disabling versioning in the registry configuration if it was enabled

If there are frequently updating registry properties, having the versioning enabled for registry resources in the registry can lead to unnecessary growth in the registry related tables in the database. To avoid this, versioning has been disabled by default from API Manager 3.0.0 onwards.

But, if registry versioning was enabled by you in WSO2 API-M 3.0.0 setup, it is **required** run the below scripts against **the database that is used by the registry**.

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
        Changing these configuration should only be done before the initial API-M Server startup. If changes are done after the initial startup, the registry resource created previously will not be available.

-   [Step 1 - Migrate the API Manager configurations](#step-1-migrate-the-api-manager-configurations)
-   [Step 2 - Upgrade API Manager to 3.1.0](#step-2-upgrade-api-manager-to-310)
-   [Step 3 - Optionally, migrate the configurations for WSO2 API-M Analytics](#step-3-optionally-migrate-the-configurations-for-wso2-api-m-analytics)
-   [Step 4 - Restart the WSO2 API-M 3.1.0 server](#step-4-restart-the-wso2-api-m-300-server)

### Step 1 - Migrate the API Manager configurations

!!! warning
    Do not copy entire configuration files from the current version of WSO2 API Manager to the new one, as some configuration files may have changed. Instead, redo the configuration changes in the new configuration files.
Follow the instructions below to move all the existing API Manager configurations from the current environment to the
 new one. For more information refer [Configuration Catalog]({{base_path}}/reference/config-catalog)

1.  Back up all databases in your API Manager instances along with the Synapse configurations of all the tenants and the super tenant.

    -   The Synapse configurations of the super tenant are in the `<OLD_API-M_HOME>/repository/deployment/server/synapse-configs/default` directory.

    -   The Synapse configurations of tenants are in the `<OLD_API-M_HOME>/repository/tenants` directory.

    -   If you use a **clustered/distributed API Manager setup** , back up the available configurations in the **API Gateway** node.

2.  Download [WUM updated](https://docs.wso2.com/display/updates/Getting+Started) pack for [WSO2 API Manager 3.1.0](http://wso2.com/api-management/).

3.  Open the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file and provide the datasource configurations for the following databases.

    -   User Store
    -   Registry database/s
    -   API Manager databases

    !!! note
        If you have used separate DBs for user management and registry in the previous version, you need to configure WSO2REG_DB and WSO2UM_DB databases separately in API-M 3.1.0 to avoid any issues.

    SHARED_DB should point to the previous API-M version's `WSO2REG_DB`. This example shows to configure MySQL database configurations.

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

    Optionally add a new entry as below to the `deployment.toml` if you have configured a seperate user management database in the previous API-M version.

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

4.   If you have used separate DB for user management in your previous setup and you have defined the `[database.user]` as mentioned in the above step, then you need to update `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file as follows, to point to the correct database for user management purposes. **Note** that the data_source name cannot be changed and it should be **WSO2USER_DB**.

    ```
    [realm_manager]
    data_source = "WSO2USER_DB"
    ```

5.  Copy the relevant JDBC driver to the `<API-M_3.1.0_HOME>/repository/components/lib` folder.


6.  Move all your Synapse configurations to API-M 3.1.0 pack.
    -   Move your Synapse super tenant configurations.
        Copy the contents in the `<OLD_API-M_HOME>/repository/deployment/server/synapse-configs/default` directory and replace the contents in the `<API-M_3.1.0_HOME>/repository/deployment/server/synapse-configs/default` directory with the copied contents.
    -   Move all your tenant Synapse configurations.
        Copy the contents in the `<OLD_API-M_HOME>/repository/tenants` directory and replace the contents in the `<API-M_3.1.0_HOME>/repository/tenants` directory with the copied contents.

    !!! warning
        When moving the Synapse configurations, **do not replace** the following set of files as they contain some modificatiosn in API-M 3.1.0 version.

        -   /api/\_RevokeAPI_.xml
        -   /sequences/\_cors_request_handler_.xml
        -   /sequences/main.xml
        -   /proxy-services/WorkflowCallbackService.xml

    !!! attention 
        If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Gateway** node.

7.  Move all your Execution plans from `<API-M_3.0.0_HOME>/repository/deployment/server/executionplans` directory to `<API-M_3.1.0_HOME>/repository/deployment/server/executionplans` directory.

    !!! note
        If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Traffic Manager** node.

8.  If you manually added any custom OSGI bundles to the `<API-M_3.0.0_HOME>/repository/components/dropins` directory, copy those to the `<API-M_3.1.0_HOME>/repository/components/dropins` directory. 

9.  If you manually added any JAR files to the `<API-M_3.0.0_HOME>/repository/components/lib` directory, copy those and paste them in the `<API-M_3.1.0_HOME>/repository/components/lib` directory.

### Step 2 - Upgrade API Manager to 3.1.0

1.  Stop all WSO2 API Manager server instances that are running.

2.  Make sure you backed up all the databases and Synapse configurations as instructed in [step 1](#step-1-migrate-the-api-manager-configurations) of the previous section.

3.  Upgrade the WSO2 API Manager database from version 3.0.0 to version 3.1.0 by executing the relevant database script, from the scripts that are provided below, on the `WSO2AM_DB` database.

    ??? info "DB Scripts"
        ```tab="DB2"
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

    **Note:** From API Manager 3.0.0 onwards, keystores could be configured as follows.
    
    ```toml
    - keystore.primary - the default keystore. If only primary is configured, the primary keystore will be used for internal and SSL related encryption and decryption tasks. 
    - keystore.internal - the keystore used for internal encryption/decrypton
    - keystore.tls - the keystore used for SSL encryption
    ```

    !!! note "If you have enabled Secure Vault"
        If you have enabled secure vault in the previous API-M version, you need to add the property values again according to the new config modal and run the script as below. Please refer [Encrypting Passwords in Configuration files]({{base_path}}/install-and-setup/setup/security/logins-and-passwords/working-with-encrypted-passwords) for more details.

        ```tab="Linux"
        ./ciphertool.sh -Dconfigure
        ```

        ```tab="Windows"
        ./ciphertool.bat -Dconfigure
        ```
    - If the truststore used in the previous setup was changed (for e.g. the name of the truststore file or the password was changed), copy the truststore to the `<API-M_3.1.0_HOME>/repository/resources/security` directory and update the credentials in the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` as follows.  
       ```toml
       [truststore]
       type= "JKS"
       file_name = "modified-client-truststore.jks"
       password= "modified_password"
       ```
       
    - In order to work with the [API Security Audit Feature]({{base_path}}/learn/api-security/configuring-api-security-audit/) you need to have the public certificate of the [42crunch](https://42crunch.com/) in the client-truststore. Follow the guidelines given in [Importing Certificates to the Truststore]({{base_path}}/install-and-setup/setup/security/configuring-keystores/keystore-basics/creating-new-keystores/#step-3-importing-certificates-to-the-truststore).
  
5.  Upgrade the Identity component in WSO2 API Manager from version 5.9.0 to 5.10.0.

    !!! note
        If you are using WSO2 Identity Server (WSO2 IS) as a Key Manager, you have to follow the instructions in [Upgrading WSO2 IS as the Key Manager to 5.10.0]({{base_path}}/install-and-setup/upgrading-wso2-is-as-key-manager/upgrading-from-is-km-590-to-5100) instead of below steps.

        But, if you are not using WSO2 IS as a Key Manager, you have to follow the below steps in order to upgrade identity components which have been shared with WSO2 API-M.

    1.  Download the identity component migration resources and unzip it in a local directory.

         Navigate to the [latest release tag](https://github.com/wso2-extensions/identity-migration-resources/releases/latest) and download the `wso2is-migration-x.x.x.zip` under Assets. Let's refer to this directory that you downloaded and extracted as `<IS_MIGRATION_TOOL_HOME>`. 

    2.  Copy the `migration-resources` folder from the extracted folder to the `<API-M_3.1.0_HOME>` directory.

    3.  Open the `migration-config.yaml` file in the migration-resources directory and make sure that the `currentVersion` element is set to 5.9.0, as shown below.

        ``` java
        migrationEnable: "true"
        currentVersion: "5.9.0"
        migrateVersion: "5.10.0"
        ```

        !!! note
            Make sure you have enabled migration by setting the `migrationEnable` element to `true` as shown above. You have to remove the following 2 steps from  migration-config.yaml which is included under version: "5.10.0".
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

    5. Update <API-M_3.1.0_HOME>/repository/conf/deployment.toml file as follows, to point to the previous user store.
    
        ```
        [user_store]
        type = "database"
        ```
    
    6.  Start WSO2 API Manager 3.1.0 as follows to carry out the complete Identity component migration.
        
        !!! note        
            If you are migrating your user stores to the new user store managers with the unique ID capabilities, Follow the guidelines given in the [Migrating User Store Managers](https://is.docs.wso2.com/en/latest/setup/migrating-userstore-managers/) before moving to the next step
                    
        ```tab="Linux / Mac OS"
        sh wso2server.sh -Dmigrate -Dcomponent=identity
        ```

        ```tab="Windows"
        wso2server.bat -Dmigrate -Dcomponent=identity
        ```

        !!! note
            Please note that depending on the number of records in the identity tables, this identity component migration will take a considerable amount of time to finish. Do not stop the server during the migration process and please wait until the migration process finish completely and server get started.

        !!! note
            Please note that if you want to use the latest user store, please update the <API-M_3.1.0_HOME>/repository/conf/deployment.toml as follows after the identity migration,

            ```
            [user_store]
            type = "database_unique_id"
            ```

        !!! warning "Troubleshooting"
            When running the above step if you encounter the following error message, please follow the steps in this section. Please note that this error could occur only if the identity tables contain a huge volume of data.

            Sample exception stack trace is given below.
            ```
            ERROR {org.wso2.carbon.registry.core.dataaccess.TransactionManager} -  Failed to start new registry transaction. {org.wso2.carbon.registry.core.dataaccess.TransactionManager} org.apache.tomcat.jdbc.pool.PoolExhaustedException: [pool-30-thread-11] Timeout: Pool empty. Unable to fetch a connection in 60 seconds, none available[size:50; busy:50; idle:0; lastwait:60000
            ```

            1.  Add the following property in `<API-M_HOME>/repository/conf/deployment.toml` to a higher value (e.g., 10)
                ```
                [indexing]
                frequency = 10
                ```

            2.  Re-run the command above.

            **Make sure to revert the change done in Step 1 , after the migration is complete.**

    7.  After you have successfully completed the migration, stop the server and remove the following files and folders.

        -   Remove the `org.wso2.carbon.is.migration-x.x.x.jar` file, which is in the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

        -   Remove the `migration-resources` directory, which is in the `<API-M_3.1.0_HOME>` directory.

        -   If you ran WSO2 API-M as a Windows Service when doing the identity component migration , then you need to remove the following parameters in the command line arguments section (CMD_LINE_ARGS) of the wso2server.bat file.

            ```
            -Dmigrate -Dcomponent=identity
            ```

6.  Migrate the API Manager artifacts.

    You have to run the following migration client to update the registry artifxacts.

    1. Download and extract the [migration-resources.zip]({{base_path}}/assets/attachments/install-and-setup/migration-resources.zip). Copy the extracted `migration-resources`  to the `<API-M_3.1.0_HOME>` folder.

    2. Download and copy the [API Manager Migration Client]({{base_path}}/assets/attachments/install-and-setup/org.wso2.carbon.apimgt.migrate.client-3.1.0-6.jar) to the `<API-M_3.1.0_HOME>/repository/components/dropins` folder.

    3.  Start the API-M server as follows.

        ``` tab="Linux / Mac OS"
        sh wso2server.sh -DmigrateFromVersion=3.0.0
        ```

        ``` tab="Windows"
        wso2server.bat -DmigrateFromVersion=3.0.0
        ```

    4. Shutdown the API-M server.
    
       -   Remove the `org.wso2.carbon.apimgt.migrate.client-3.1.0-6.jar` file, which is in the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

       -   Remove the `migration-resources` directory, which is in the `<API-M_3.1.0_HOME>` directory.

7.  Re-index the artifacts in the registry.
    1.  Run the [reg-index.sql]({{base_path}}/assets/attachments/install-and-setup/reg-index.sql) script against the `SHARED_DB` database.

        !!! note
            Please note that depending on the number of records in the REG_LOG table, this script will take a considerable amount of time to finish. Do not stop the execution of script until it is completed.

    2.  Add the [tenantloader-1.0.jar]({{base_path}}/assets/attachments/install-and-setup/tenantloader-1.0.jar) to `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

        !!! attention
            If you are working with a **clustered/distributed API Manager setup**, follow this step on the **Store and Publisher** nodes.

        !!! note
            You need to do this step, if you have **multiple tenants** only.

    3.  Add the following configuration in to `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file.
        
        ```
        [indexing]
        re_indexing = 1
        ```

        !!! info 
             If you use a clustered/distributed API Manager setup, do the above change in deployment.toml of Publisher and Devportal nodes
             
    4.  If the `<API-M_3.1.0_HOME>/solr` directory exists, take a backup and thereafter delete it.

    5.  Start the WSO2 API-M server.

    6.  Stop the WSO2 API-M server and remove the `tenantloader-1.0.jar` from the `<API-M_3.1.0_HOME>/repository/components/dropins` directory.

### Step 3 - Optionally, migrate the configurations for WSO2 API-M Analytics

!!! warning
    This step is **only required** if you have WSO2 API-M-Analytics configured in your current deployment.

Follow the steps below to migrate APIM Analytics 3.0.0 to APIM Analytics 3.1.0

#### Step 3.1 - Migrating the Analytics Database

Upgrade the WSO2 API Manager Analytics database from version 3.0.0 to version 3.1.0 by executing the relevant database script, from the scripts that are provided below, on the `APIM_ANALYTICS_DB` database.

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

#### Step 3.2 - Configure WSO2 API-M Analytics 3.1.0

!!! info
    Sometimes due to case insensitivity of primary keys in aggregation tables, primary key violation errors are thrown when you try to insert a new record with the same value as an existing one. To overcome this, you need to add encoding and collation to database when the Analytics DB is created (i.e., before the tables are created). For more information on collation, see [MySQL](https://dev.mysql.com/doc/refman/5.7/en/charset-collation-names.html) or [MS SQL](https://docs.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-ver15) based on the database that you are using. Sample commands are provided below.

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

    !!! info "To convert the jar files to OSGi bundles, follow the steps given below."
        1. Download the non-OSGi jar for the required third party product, and save it in a preferred directory in your machine.
        2. Go to the `<API-M_ANALYTICS_HOME>/bin` directory. Run the command given below, to generate the converted file in the `<API-M_ANALYTICS_HOME>/lib` directory.

        ```
        ./jartobundle.sh <PATH_TO_NON-OSGi_JAR> ../lib
        ```

5.  Start the Worker and Dashboard profiles as below by navigating to `<API-M_ANALYTICS_3.1.0_HOME>/bin` location.
    
    ```tab="Worker"
    sh worker.sh
    ```

    ```tab="Dashboard"
    sh dashboard.sh
    ```

!!! note
    If you have developed any custom dashboards in API-M 3.0.0 Analytics using Stream Processor, you will be able to use the same in API-M Anaytics 3.1.0 as well. If you require any guidance regarding this, you can contact [WSO2 Support](https://support.wso2.com/jira/secure/Dashboard.jspa).

#### Step 3.3 - Configure WSO2 API-M 3.1.0 for Analytics

Enable analytics in WSO2 API-M by setting the following configuration to true in the `<API-M_3.1.0_HOME>/repository/conf/deployment.toml` file.

    ``` java
    [apim.analytics]
    enable = true
    ```

### Step 4 - Restart the WSO2 API-M 3.1.0 server

1.  Restart the WSO2 API-M server.

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

This concludes the upgrade process.

!!! tip
    The migration client that you use in this guide automatically migrates your tenants, workflows, external user stores, etc. to the upgraded environment. Therefore, there is no need to migrate them manually.
