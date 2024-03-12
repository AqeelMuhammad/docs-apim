# Configuring APIM Analytics

WSO2 API Manager Analytics provides reports, statistics, and graphs on the APIs deployed in WSO2 API Manager. You can configure alerts to monitor these APIs and detect unusual activity, manage locations via Geo-location statistics, and carry out a detailed analysis of the logs. WSO2 API Manager has an enhanced distribution of Analytics to cater to the API Manager specific scenarios that are used here to configure API-M Analytics.

To set up analytics for quick demos and try-out scenarios, see the  [**Quick Setup**](#quick-setup) section, or to set up analytics for a production environment, see the [**Standard Setup**](#standard-setup) section.

!!! note
     -   It is not recommended to configure the Gateway node as an analytics server.


-   [**Quick Setup**](#quick-setup)
-   [**Standard Setup**](#standard-setup)

## Quick Setup

Follow the instructions below if you wish to set up API-M Analytics for quick demos and to try-out scenarios.

1.  Download and install WSO2 API-M.

    WSO2 API-M via the [WSO2 API Manager page](https://wso2.com/api-management/install/). For more information on installing WSO2 API-M, see the [Installation Guide]({{base_path}}/install-and-setup/install/installation-prerequisites).
    
     [![apim download page]({{base_path}}/assets/img/learn/apim-download-page.png)]({{base_path}}/assets/img/learn/apim-download-page.png)
    
2.  Download and install WSO2 API-M Analytics.

    To download WSO2 API-M Analytics go to the [WSO2 API Manager page](https://wso2.com/api-management/install/), click **Download** to expand the installation options.Navigate to the **Other Resources** section, and click **Analytics**.

    ![]({{base_path}}/assets/img/learn/analytics-download-page.png)
    
       <html><div class="admonition warning">
       <p class="admonition-title">Note</p>
       <p>If you are following the quick setup make sure both the binaries (unzipped API-M pack and unzipped Analytics pack) are inside the same directory. 
                        Because the default configurations such as database connection urls etc are configured assuming that both the packs are inside the same folder.
                        </p>
                        ![]({{base_path}}/assets/img/learn/analytics-quick-setup.png)
       </div>
       </html>
                  
3.  To enable Analytics, open the `<API-M_HOME>/repository/conf/deployment.toml` file and uncomment the analytics enabling section as shown below. Save this change.

    ``` toml
      [apim.analytics]
      enable = true
    ```

    !!! note
          If you are working on a distributed (clustered) setup of WSO2 API Manager, do the configurations instructed to be done in API Manager in the Publisher, Store, and Gateway nodes as well.


4.  Start the Worker profile of the Analytics Server.
    <br/>Navigate to the `<API-M_ANALYTICS_HOME>/bin` directory in your console and execute one of the following scripts based on your OS.
    -   On Windows: `worker.bat --run              `
    -   On Linux/Mac OS: `sh worker.sh               `

5.  Start the API Manager server.
    <br/>Navigate to the `<API-M_HOME>/bin` directory in your console and execute one of the following scripts based on your OS.
    -   On Windows: `wso2server.bat --run`
    -   On Linux/Mac OS: `sh wso2server.sh` 
    
        !!! info
                  If API-M Analytics is properly configured in WSO2 API Manager, when you start up the API Manager server, which is after the WSO2 API-M Analytics server, you will see the following log message in the terminal that is running the API-M Analytics server.
            
                    INFO {org.wso2.carbon.databridge.core.DataBridge} - user admin connected   
                    
6.  Start the Dashboard profile of the Analytics Server.
    <br/>Navigate to the `<API-M_ANALYTICS_HOME>/bin` directory in your console and execute one of the following scripts based on your OS.
    -   On Windows: `dashboard.bat --run`  
    -   On Linux/Mac OS: `sh dashboard.sh`    
    
    

7.  Optionally, if you wish to access the business rules via the Dashboard node, you can use the dashboard profile that we started in the previous step.            

You can now start using the WSO2 API Manager for its usual operations and the required Analytics functionality.

## Standard Setup

![]({{base_path}}/assets/img/learn/analytics-standard-architecture-diagram.png)

Follow the instructions below if you wish to set up API-M Analytics for a production environment.

-   [Step 1 - Download and install WSO2 API-M](#step-1-download-and-install-wso2-api-m)
-   [Step 2 - Download and install WSO2 API-M Analytics](#step-2-download-and-install-wso2-api-m-analytics)
-   [Step 3 - Configure WSO2 API Manager to publish statistics](#step-3-configure-wso2-api-manager-to-publish-statistics)
-   [Step 4 - Configure databases](#step-4-configure-databases)
-   [Step 5 - Configure APIM IdP Client](#step-5-configure-apim-idp-client)
-   [Step 6 - Configure keystores](#step-6-configure-keystores)
-   [Step 7 - Configure User-Agent Parser](#step-7-configure-user-agent-parser)

### Step 1 - Download and install WSO2 API-M

 Download and install WSO2 API-M via the [WSO2 API Manager page](https://wso2.com/api-management/install/). Click **DOWNLOAD** and go to **INSTALLATION OPTIONS**.
 <br/>For more information on installing WSO2 API-M, see the [Installation Guide]({{base_path}}/install-and-setup/install/installation-prerequisites.md).
    
 ![]({{base_path}}/assets/img/learn/apim-download-page.png)
    
### Step 2 - Download and install WSO2 API-M Analytics

To download WSO2 API-M Analytics go to the [WSO2 API Manager page](https://wso2.com/api-management/install/), click **Download** to expand the installation options.Navigate to the **Other Resources** section, and click **Analytics**. 
<br/>As the process of installing API-M Analytics is identical to installing WSO2 Enterprise Integrator 7.0.x - Streaming Integrator (WSO2 SI), for more information go to the [WSO2 SI documentation](https://ei.docs.wso2.com/en/latest/streaming-integrator/setup/installing-si-in-vm)

  ![]({{base_path}}/assets/img/learn/analytics-download-page.png)
    
### Step 3 - Configure WSO2 API Manager to publish statistics

Follow the instructions below to do the required configurations for WSO2 API-M to publish statistics in the WSO2 API-M Analytics server.

!!! warning
      If you are working on a distributed (clustered) setup of API Manager, carryout the instructed configurations in the Publisher, Store and Gateway nodes of the API Manager.

Open the `<API-M_HOME>/repository/conf/deployment.toml` file and uncomment the following section as shown below. Save this change.

   ``` toml
   [apim.analytics]
   enable = true
   ```
Configure the following parameters under the `[apim.analytics]` section if required.

<table>
<thead>
<tr class="header">
<th>Parameter</th>
<th>Value</th>
<th>Description</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td width="10%"><code>receiver_urls</code></td>
<td width="30%">
<code>
&lt;protocol&gt;://&lt;hostname&gt;:&lt;port&gt;/
</code>
</td>
<td>
The server URL of the remote WSO2 API-M Analytics server used to collect statistics from WSO2 API Manager. Use this configuration if there is only one group with either one or many receivers.
<p> e.g., Three receivers within a single group can be specified as follows.
<br/><code>receiver_urls = tcp://localhost:7612,tcp://localhost:7613,tcp://localhost:7614</code>
</p>
<html><div class="admonition warning">
<p class="admonition-title">From DAS to SI</p>
<p>
When using WSO2 API-M Analytics 3.1.0, when publishing to an minimum HA setup of APIM analytics, you need to separate the Receiver URLs by using a pipe symbol (|) because the analytics events are published in a failover manner where only one node handles the processing at any given time.
<br/>e.g.,
<br/><code>receiver_urls = "tcp://localhost:7612 | tcp://localhost:7613"</code>
</p>
<html><div class="admonition info">
<p class="admonition-title">Note</p>
<p>If you are setting up APIM analytics on an Active-active HA setup, you need to define the URLs with <b>comma</b> separations, as the setup is configured with load-balanced configurations.
<br/><br/>
When you are configuring APIM analytics on an Active-passive HA setup, you should separate the URLs using the <b>pipe symbol (|)</b>, as the setup is configured with failover configurations.
</p>
</div>
</html>
</div>
</html>
<html><div class="admonition info">
<p class="admonition-title">Note</p>
<p>If the <code>receiver_urls</code> property is configured in the <code>[apim.analytics]</code> section, you do not need to configure the  
   <code>[[apim.analytics.url_group]]</code> properties. Configuring either one of these properties is enough.
</p>
</div>
</html>
</td>
</tr>
<tr class="odd">
<td><code>[[apim.analytics.url_group]]
<br/>analytics_url</code></td>
<td><code>&lt;protocol&gt;://&lt;hostname&gt;:&lt;port&gt;/</code></td>
<td>
An event can also be published to multiple receiver groups, where each group has one or more receivers. 
For each receiver groups we need to repeat the above section, whereas receivers are delimited by commas.
<p>
e.g., Two receiver groups with two load balanced receivers in each can be specified as follows.
<br />
<code>
<br/>[[apim.analytics.url_group]]
<br/>analytics_url =["tcp://localhost:7612","tcp://localhost:7613"]
<br/>type = "loadbalance"
<br/>
<br />[[apim.analytics.url_group]]
<br />analytics_url =["tcp://localhost:7712","tcp://localhost:7713"]
<br/>type = "loadbalance"
</code>
<br/>
<br/>If the type is not specified it defaults to the fail over.
<br/><code>
<br/>[[apim.analytics.url_group]]
<br/>analytics_url =["tcp://localhost:7612"|"tcp://localhost:7613"]
<br/>type = "failover"
<br/>
<br />[[apim.analytics.url_group]]
<br />analytics_url =["tcp://localhost:7712"|"tcp://localhost:7713"]
<br/>type = "failover"
<code>
</p>
<html><div class="admonition info">
<p class="admonition-title">Note</p>
<p>If the <code>[[apim.analytics.url_group]]</code> property group is configured in the <code>[apim.analytics]</code> section, you do no need to configure the  <code>receiver_urls</code> properties. Configuring either one of these properties is enough.
</p>
</div>
</html>
</td>
</tr>
<tr class="even">
<td><code>receiver_username</code></td>
<td>A valid administrator username</td>
<td><div class="content-wrapper">
<p>The administrator user name to log into the remote WSO2 API-M Analytics server that collects statistics from WSO2 API Manager.</p>
<p>If you enable email user, you need to configure <code>@carbon.super</code> to the username of the API-M Analytics admin user. 
<br/>e.g., If the username of the API-M Analytics admin user is <code>demo@wso2.com</code>, it must be <code>demo@wso2.com@carbon.super</code> once you have enabled email user.</li>
</p>
<html><div class="admonition info">
<p class="admonition-title">Note</p>
<p>If the username and password of the API-M Analytics node is same as that of API-M node, you can uncomment 
   <code>username = "$ref{super_admin.username}"</code> and <code>password = "$ref{super_admin.password}"</code> under 
   [apim.analytics].
   <br/>
   <br/>The default value <code>$ref{super_admin.username}</code> and <code>$ref{super_admin.password}</code> 
   retrieve the admin user and password respectively from the current API-M node and set those for <code>receiver_username</code> 
   and <code>receiver_password</code>. 
</div>
</html>
<html><div class="admonition warning">
<p class="admonition-title">username and password parameters</p>
<ul>
Please note that if the <code>username</code> and <code>password</code> are uncommented, then those values will be assigned to all the usernames and passwords that are under the [apim.analytics] section.
<li>For example if <code>store_api_username</code> is not defined, then it takes the value of <code>username</code> when <code>username</code> is uncommented.</li>
</ul>
</div>
</html>
</div></td>
</tr>
<tr class="odd">
<td><code>receiver_password</code></td>
<td>The password of the username specified.</td>
<td><div class="content-wrapper">
<p>The administrator password to log into the remote WSO2 API-M Analytics server that collects statistics from WSO2 API Manager.</p>
</div></td>
</tr>
<tr class="even">
<td><code>store_api_url</code></td>
<td><code>https://&lt;host&gt;:&lt;port&gt;</code></td>
<td>The WSO2 API-M Analytics REST API URL. The WSO2 API-M Analytics REST API connection information, which are included under the REST API-M connection information, are defined as global properties, as they are common to all the WSO2 API-M analytics.</td>
</tr>
<tr class="odd">
<td><code>store_api_username </code></td>
<td>A valid administrator username</td>
<td>The administrator username to log into the remote WSO2 API-M Analytics server.</td>
</tr>
<tr class="even">
<td><code>store_api_password</code></td>
<td>The password of the username specified.</td>
<td>The administrator password to log into the remote WSO2 API-M Analytics server.</td>
</tr>
</tbody>
</table>

Save the changes.

### Step 4 - Configure databases

Configuring databases allow you to persist data relating to APIs, process them and analyze. Follow the procedure below to configure databases. 

!!! note
    If you are configuring APIM Analytics related databases in **Oracle**, apart from the below mentioned configurations, 
    you need to add the `alter session set NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS'` section to each analytics related 
    database(either in worker or dashboard) you configure with Oracle.
    
    In the following example `APIM_ANALYTICS_DB` is configured with Oracle.
    ``` java
    - name: APIM_ANALYTICS_DB
      description: "The datasource used for APIM statistics aggregated data."
      jndiConfig:
        name: jdbc/APIM_ANALYTICS_DB
        definition:
          type: RDBMS
          configuration:
            jdbcUrl: 'jdbc:oracle:thin:@localhost:1521:XE'
            username: 'root'
            password: '123'
            driverClassName: oracle.jdbc.OracleDriver
            minIdle: 5
            maxPoolSize: 50
            idleTimeout: 60000
            connectionTestQuery: SELECT 1 FROM DUAL
            connectionInitSql: alter session set NLS_DATE_FORMAT='YYYY-MM-DD HH24:MI:SS'
            validationTimeout: 30000
            isAutoCommit: false
    ```
#### Configure Analytics

!!! warning
    When configuring API-M Analytics, change only the required properties(which are mentioned in the respective documentation) 
    of the deployment.yaml file of either worker or dashboard runtime. Do not change other default values as it may result in an erroneous state.

API-M Analytics contains two runtime, namely worker and dashboard. The worker is responsible for the summarization of the collected data and the dashboard is responsible to represent the summarised data in the dashboards. Therefore, two separate JVMs are required. As a best practice, the worker and dashboard runtime can have the same analytics binary. This helps when managing the deployment and when applying WUM updates. However, it is up to the dev-ops engineer to decide whether to use the same binary (pack) or two binaries for the two runtime.
    
The Worker supports an [Active-Active]({{base_path}}/install-and-setup/setup/distributed-deployment/configure-apim-analytics/active-active/) deployment and an [Active-Passive]({{base_path}}/install-and-setup/setup/distributed-deployment/configure-apim-analytics/active-passive/) deployment. 

As the dashboard is used only to render the data there is no Active-Active or Active-Passive concept. However, based on the High Availability (HA) requirement it can be configured as Active-Active or Active-Passive by defining the load balancer related configurations. 

!!! info
    Sometimes due to case insensitivity of primary keys in aggregation tables, primary key violation errors are thrown when you try to insert a new record with the same value as an existing one. To overcome this, you need to add encoding and collation to database when the Analytics DB is created (i.e., before the tables are created). For more information on collation, see [MySQL](https://dev.mysql.com/doc/refman/5.7/en/charset-collation-names.html) or [MS SQL](https://docs.microsoft.com/en-us/sql/relational-databases/collations/collation-and-unicode-support?view=sql-server-ver15) based on the database that you are using. Sample commands are provided below.

    !!! example

        ```sql tab="MySQL"
        ALTER DATABASE <DB-NAME> COLLATE latin1_general_cs ;
        ```
        
        ```sql tab="MS SQL"
        ALTER DATABASE <DB-NAME> COLLATE SQL_Latin1_General_CP1_CS_AS ;
        ```

1.  Stop the WSO2 API-M Analytics server if it is running already.
2.  Configure the dashboard profile.
    1. Open the `<API-M_ANALYTICS_HOME>/conf/dashboard/deployment.yaml` file.
    2. Edit the `APIM_ANALYTICS_DB` and `AM_DB` sections and point to your desired type of database. 
       <br/>A sample for MySQL is shown below.
       
        !!! note
            In the below configuration, the database defined as `am_db` is the same database which is defined under
             `[database.apim_db]` configuration in the **deployment.toml** file of the WSO2 API Manager.

         ``` java
           - name: AM_DB
               description: Main datasource used by API Manager
               jndiConfig:
                 name: jdbc/AM_DB
               definition:
                 type: RDBMS
                 configuration:
                   jdbcUrl: "jdbc:mysql://localhost:3306/am_db"
                   username: wso2carbon
                   password: wso2carbon
                   driverClassName: com.mysql.jdbc.Driver
                   maxPoolSize: 10
                   idleTimeout: 60000
                   connectionTestQuery: SELECT 1
                   validationTimeout: 30000
                   isAutoCommit: false

         ```
    
3.  Configure the worker profile. 
    
      1. Open the `<API-M_ANALYTICS_HOME>/conf/worker/deployment.yaml` file 
      2. Edit the `APIM_ANALYTICS_DB` section. 
       <br/>A sample for MySQL is shown below.

          ``` java
            - name: APIM_ANALYTICS_DB
                 description: "The datasource used for APIM statistics aggregated data."
                 jndiConfig:
                   name: jdbc/APIM_ANALYTICS_DB
                 definition:
                   type: RDBMS
                   configuration:
                     jdbcUrl: 'jdbc:mysql://localhost:3306/stats_db'
                     username: 'root'
                     password: '123'
                     driverClassName: com.mysql.jdbc.Driver
                     minIdle: 5
                     maxPoolSize: 50
                     idleTimeout: 60000
                     connectionTestQuery: SELECT 1
                     validationTimeout: 30000
                     isAutoCommit: false
          ```
       
4.  Point the following data sources to external databases. 
       None of the following databases need DB scripts. The tables will be automatically created.
      1. WSO2_DASHBOARD_DB (dashboard profile)
      2. BUSINESS_RULES_DB (dashboard profile)
      3. WSO2_PERMISSIONS_DB (worker + dashboard)
      4. GEO_LOCATION_DATA (Only if you need geo-location based statistics.)
5.  Start the WSO2 API-M Analytics server.

!!! Info
      **Do the following to integrate third party products when configuring databases.**
      <br/>WSO2 API Manager Analytics is a OSGi-based product. Therefore, when you integrate third party products such as Oracle with WSO2 API-M Analytics, you need to check whether the libraries you need to add are OSGi based. If they are not, you need to convert them to OSGi bundles before adding them to the `<API-M_ANALYTICS_HOME>/lib` directory.
      For detailed instructions, see [Adding Third Party Non OSGi Libraries]({{base_path}}/learn/analytics/adding-third-party-non-osgi-libraries/).

### Step 5 - Configure APIM IdP Client

APIM IdP Client authenticates users by interacting with the identity provider of API Manager via OAuth2. The APIM Manager user store is used to provide the access to APIM Analytics as well. WSO2 APIM Analytics server authenticates by requesting an access token from the identity provider in API Manager using the authentication code grant type. This APIM IdP client enables SSO(Single Sign On). 

Furthermore, APIM IdP client functionality can be controlled via the properties defined in the <APIM_ANALYTICS_HOME>/conf/dashboard/deployment.yaml file under the auth.configs namespace as shown below.

```
auth.configs:
  type: apim
  ssoEnabled: true
  properties:
    adminScope: apim_analytics:admin_carbon.super
    allScopes: apim_analytics:admin apim_analytics:product_manager apim_analytics:api_developer apim_analytics:app_developer apim_analytics:devops_engineer apim_analytics:analytics_viewer apim_analytics:everyone openid apim:api_view apim:subscribe
    adminUsername: admin
    adminPassword: admin
    kmDcrUrl: https://localhost:9443/client-registration/v0.16/register
    kmTokenUrlForRedirection: https://localhost:9443/oauth2
    kmTokenUrl: https://localhost:9443/oauth2
    kmUsername: admin
    kmPassword: admin
    portalAppContext: analytics-dashboard
    businessRulesAppContext : business-rules
    cacheTimeout: 30
    baseUrl: https://localhost:9643
    grantType: authorization_code
    publisherUrl: https://localhost:9443
    devPortalUrl: https://localhost:9443
    externalLogoutUrl: https://localhost:9443/oidc/logout
```

#### Details of the properties in APIM IdP Client


|**Property**                                           |**Default Value**                    |**Description**                 |
|-------------------------------------------------------|-------------------------------------|--------------------------------|
| `adminScope`| apim_analytics:admin_carbon.super | Admin scope which is used for permissions in dashboards.|
| `allScopes`| apim_analytics:admin apim_analytics:product_manager apim_analytics:api_developer apim_analytics:app_developer apim_analytics:devops_engineer apim_analytics:analytics_viewer apim_analytics:everyone openid apim:api_view apim:subscribe | All the scopes used for permissions in the dashboards.|
| `adminUsername`| admin | The username for the admin services that corresponds to the credentials defined under the super tenant in the API Manager's `deployment.toml` file.|
| `adminPassword`| admin | The password for the admin services that corresponds to the credentials defined under the super tenant in the API Manager's `deployment.toml` file.|
| `kmDcrUrl`| https://localhost:9443/client-registration/v0.16/register | The Dynamic Client Registration (DCR) endpoint of the key manager in the IdP. This should be pointed to the API Manager Publisher node url.|
| `kmTokenUrlForRedirection`| https://localhost:9443/oauth2 | The token endpoint of the key manager in the IdP which is used for browser redirection. This should be pointed to the API Manager Publisher node url.|
| `kmTokenUrl`| https://localhost:9443/oauth2 | The token endpoint of the key manager in the IdP. This should be pointed to the API Manager Publisher node url.|
| `kmUsername`| admin | The username for the key manager in the IdP.|
| `kmPassword`| admin | The password for the key manager in the IdP.|
| `portalAppContext`| analytics-dashboard | The application context of the Analytics Dashboard application.|
| `businessRulesAppContext`| business-rules| The application context of the Business Rules application.|
| `cacheTimeout`| 30 | The cache timeout for the validity period of the token in seconds.|
| `baseUrl`| https://localhost:9643 | The base URL to which the token should be redirected after the code returned from the Authorization Code grant type is used to get the token. This is the URL where the API-M Analytics Dashboard server is running.|
| `grantType`| authorization_code | 	The grant type used in the OAuth application token request.|
| `publisherUrl`| https://localhost:9443 | Url which the API Manager Publisher is running.|
| `devPortalUrl`| https://localhost:9443 | Url which the API Manager Developer Portal  is running.|
| `externalLogoutUrl`| https://localhost:9443/oidc/logout | The URL via which you can log out from the external IDP provider(API Manager) side in the SSO. This should be pointed to the API Manager Publisher node url.|

### Step 6 - Configure keystores

In the SSL handshake between the API Manager and API Manager Analytics servers, the client (i.e. API Manager) needs to verify the certificate presented by the server (i.e. API Manager Analytics). For this purpose, the client stores the trusted certificate of the server in the `client-truststore.jks` keystore.

If you use a custom keystore in API Manager and/or API Manager Analytics, import the public key certificate of API Manager Analytics into the `client-truststore.jks` file of the API Manager. To export the public key from the server and import it into the client's trust store, see [Adding CA-signed certificates to keystores]({{base_path}}/install-and-setup/setup/security/configuring-keystores/keystore-basics/creating-new-keystores/#adding-ca-signed-certificates-to-keystores).

For more information follow [Configuring Keystores in APIM Analytics]({{base_path}}/learn/analytics/configuring-keystores-in-apim-analytics/).

### Step 7 - Configure User-Agent Parser

The User-Agent and Operating System information is extracted from the User-Agent header of the API requests for the purpose of analytics. This process requires a set of regular expressions to parse the header and extract the information. By default, it is configured to use the `<API-M_ANALYTICS_HOME>/conf/worker/regexs.yaml` file for this purpose as shown below.

```
siddhi:
  extensions:
    # Provides the regular expression collection to parse the user-agent header
    -
      extension:
        name: 'getUserAgentProperty'
        namespace: 'env'
        properties:
          regexFilePath : ${sys:carbon.home}/conf/worker/regexes.yaml
```
However, if you need to use your own regular expressions to extract the information in detail, then you can replace the `regexFilePath` property with your own file.

!!! warning
    The regular expressions configured above is reduced to provide the optimal performance, while identifying common User-Agents and Operating Systems. However, if you completely remove the latter mentioned configuration, you will end-up by using a standard regular expression set, which is packed inside the parser library, and it will extract almost every User-Agents and Operating Systems, but might provide lower performance throughput.
