# Analytics for Choreo Connect

Choreo Connect Analytics provides reports, dashboards, statistics, and graphs for the APIs deployed on Choreo Connect.
WSO2 Choreo Connect has the capability to publish events to the Choreo platform in order to generate analytics. This page describes the feature and explains how the feature could be used to generate useful analytics in order to gain important insights into the APIs deployed on the Choreo Connect. To learn more concepts on analytics, follow the [concepts]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/concepts/choreo-connect-analytics/).


## Configuring Analytics for Choreo Connect

The following steps will describe how to configure Choreo Connect Analytics with Choreo portal.

!!! note 
    Before you begin, make sure to go through [main configurations]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/configurations/configuration-overview/) and [Configurations for Analytics]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/configurations/analytics-configurations/) and familiar with the configurations.

### Step 1 - Set up Analytics

To configure analytics with Choreo,

1. Sign up to <a href="https://console.choreo.dev/login/" onclick="appInsights.trackEvent({name: 'apim-docs-choreo-connect-setup-api-analytics'});">https://console.choreo.dev/login/</a>.
2. Go to <a href="https://console.choreo.dev/user-settings/onpremkeys" onclick="appInsights.trackEvent({name: 'apim-docs-choreo-connect-setup-api-analytics-on-prem-key'});">https://console.choreo.dev/user-settings/onpremkeys</a> and generate a on-prem-key.
3. Open the `docker-compose.yaml` file located in the `CHOREO-CONNECT_HOME/docker-compose/choreo-connect` or `CHOREO-CONNECT_HOME/docker-compose/choreo-connect-with-apim` based on your choice on the setup.

    !!! info
        Choreo Connect can be configured to pulish Analytics to Choreo cloud in both [standalone mode]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/getting-started/deploy/cc-as-a-standalone-gateway-on-docker/#step-3-start-choreo-connect) and [with Control Plane mode]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/getting-started/quick-start-guide-docker-with-apim/#step-3-start-choreo-connect-and-api-manager).

4. Find the environment variables section under the `enforcer` and change the below variables.

    ``` yml
    environment:
        ...
        analytics_authURL=https://analytics-event-auth.choreo.dev/auth/v1
        analytics_authToken=<your-on-prem-key>
    ```

5. Now open the `config.toml` located in `CHOREO-CONNECT_HOME/docker-compose/choreo-connect/conf` directory and find the [Analytics]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/configurations/analytics-configurations/) section and enable analytics by using the following configurations.

    ``` toml
    [analytics]
    enabled = true
    [analytics.adapter]
        bufferFlushInterval = "1s"
        bufferSizeBytes = 16384
        gRPCRequestTimeout = "20s"
    [analytics.enforcer]
        [analytics.enforcer.configProperties]
        authURL = "$env{analytics_authURL}"
        authToken = "$env{analytics_authToken}"

        [analytics.enforcer.LogReceiver]
        port = 18090
        maxMessageSize = 1000000000
        maxHeaderLimit = 8192
        #keep alive time of the external authz connection
        keepAliveTime = 600

        [analytics.enforcer.LogReceiver.threadPool]
            coreSize = 10
            maxSize = 100
            #keep alive time of threads in seconds
            keepAliveTime = 600
            queueSize = 1000
    ```

### Step 2 - Try it out

Let's generate some traffic to see the Analytics in Choreo cloud.

1. Deploy your API - Follow [this]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/deploy-api/deploy-rest-api-in-choreo-connect/#step-3-deploy-the-api-in-api-manager) according to your setup.

2. Let's Invoke the API few times - [Invoke the API]({{base_path}}/deploy-and-publish/deploy-on-gateway/choreo-connect/deploy-api/deploy-rest-api-in-choreo-connect/#step-4-invoke-the-api)

3. Go to <a href="https://console.choreo.dev/insights" onclick="appInsights.trackEvent({name: 'apim-docs-choreo-connect-setup-api-analytics-choreo-insights'});">Choreo Insights</a>  to view statistics. 

Here are some of the graphs generated in Choreo cloud.

[![Choreo Analytics Overview]({{base_path}}/assets/img/deploy/choreo-analytics-overview.png)]({{base_path}}/assets/img/deploy/choreo-analytics-overview.png)

[![Choreo Analytics Traffic]({{base_path}}/assets/img/deploy/choreo-analytics-traffic.png)]({{base_path}}/assets/img/deploy/choreo-analytics-traffic.png)

[![Choreo Analytics Latency]({{base_path}}/assets/img/deploy/choreo-analytics-latency.png)]({{base_path}}/assets/img/deploy/choreo-analytics-latency.png)

## See Also

- [Publishing Analytics Events to External Systems]({{base_path}}/api-analytics/samples/publishing-analytics-events-to-external-systems)