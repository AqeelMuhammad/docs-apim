#  Configuring Environment Specific Parameters

When there are multiple environments, to allow easily configuring environment-specific details, apictl supports an additional parameter file named `api_params.yaml`. It is recommended to store the parameter file with the API Project; however, it can be stored anywhere as required. 

After the file is placed in the project directory, the tool will auto-detect the parameters file upon running the `import-api` command and create an environment-based artifact for API Manager. If the `api_params.yaml` is not found in the project directory, the tool will lookup in the project’s base path and the current working directory. 

The following is the structure of the parameter file.

```go
environments:
    - name: <environment_name>
      endpoints:
          production:
              url: <production_endpoint_url>
              config:
                  factor: <suspension_factor>
                  suspendMaxDuration: <maximum_suspend_time_duration>
                  suspendDuration: <suspend_time_duration>
                  suspendErrorCode: 
                      - <suspend_error_code_1>
                      - <suspend_error_code_2>
                      - ......................
                      - <suspend_error_code_n>
                  retryTimeOut: <no_of_retries_before_suspension>
                  retryDelay: <retry_delay_in_ms>
                  retryErroCode:
                      - <retry_error_code_1>
                      - <retry_error_code_2>
                      - ....................
                      - <retry_error_code_n>
                  actionSelect: <action_discard_or_fault>
                  actionDuration: <action_time_duration>
          sandbox:
              url: <sandbox_endpoint_url>
              config:
                  factor: <suspension_factor>
                  suspendMaxDuration: <maximum_suspend_time_duration>
                  suspendDuration: <suspend_time_duration>
                  suspendErrorCode: 
                      - <suspend_error_code_1>
                      - <suspend_error_code_2>
                      - ......................
                      - <suspend_error_code_n>
                  retryTimeOut: <no_of_retries_before_suspension>
                  retryDelay: <retry_delay_in_ms>
                  retryErroCode:
                      - <retry_error_code_1>
                      - <retry_error_code_2>
                      - ....................
                      - <retry_error_code_n>
                  actionSelect: <action_discard_or_fault>
                  actionDuration: <action_time_duration>
      security:
          enabled: <whether_security_is_enabled>
          type: <endpoint_authentication_type_basic_or_digest>
          username: <endpoint_username>
          password: <endpoint_password>
      gatewayEnvironments:
          - <gateway_environment_name>           
      certs:
          - hostName: <endpoint_url>
            alias: <certificate_alias>
            path: <certificate_file_path>
      mutualSslCerts:
          - tierName: <subscription_tier_name>
            alias: <certificate_alias>
            path: <certificate_file_path>
	  policies:
          - <subscription_policy_1_name>
          - <subscription_policy_2_name>
```
The following code snippet contains sample configuration of the parameter file.

!!! example
    ```go
    environments:
        - name: dev
          endpoints:
              production:
                  url: 'https://dev.wso2.com'
          security:
              enabled: true
              type: basic
              username: 'admin'
              password: 'admin'
          certs:
              - hostName: 'https://dev.wso2.com'
                alias: Dev
                path: ~/.certs/dev.pem 
          gatewayEnvironments:
              - Production and Sandbox
          policies:
              - Gold
              - Silver
        - name: test
          endpoints:
              production:
                  url: 'https://test.wso2.com'
                  config:
                      factor: 3
                      suspendMaxDuration: 25000
                      suspendDuration: 45000
                      suspendErrorCode: 
                          - "101504"
                          - "101501"
                      retryTimeOut: $RETRY
                      retryDelay: 23000
                      retryErroCode:
                          - "101503" 
                          - "101504"
                      actionSelect: discard
                      actionDuration: 75000
              sandbox:
                  url: 'https://test.sandbox.wso2.com'
          security:
              enabled: true
              type: digest
              username: 'admin'
              password: 'admin'
        - name: production
          endpoints:
            production:
                  url: 'https://prod.wso2.com'
            mutualSslCerts:
                - tierName: Unlimited
                  alias: Prod1
                  path: ~/.certs/prod1.crt
                - tierName: Gold
                  alias: Prod2
                  path: ~/.certs/prod2.crt
    ```

!!! warning
    [Configuring OAuth 2.0 Endpoint Security]({{base_path}}/learn/api-controller/advanced-topics/configuring-different-endpoint-security-types/#configuring-oauth-20-endpoint-security) using the parameters file has been introduced via an U2/WUM update and is effective from 8th October 2021 (2021-10-08).  
    
    For more information on how to update using U2, see [Updates 2.0 Documentation](https://updates.docs.wso2.com/en/latest/updates/overview/). For more information on how to update using WUM, see the documentation [Using WSO2 Update Manager](https://docs.wso2.com/display/updates100/Using+WSO2+Update+Manager).

Instead of the default `api_params.yaml`, you can provide a custom parameter file using `--params` flag. A sample command will be as follows.

!!! example
    ```go
    apictl import-api -f dev/PhoneVerification_1.0.zip -e production --params /home/user/custom_params.yaml 
    ```

!!! info
    -   Production/Sandbox backends for each environment can be specified in the parameter file with additional configurations, such as timeouts.
    -   Quotes are mandatory if `password` contains special characters.
    -   Certificates (Endpoint certificates and MutualSSL certificates) for each URL can be configured in the parameter file. For certificates, a valid path to the certificate file is required. 
    -   The parameter file supports detecting environment variables during the API import process. You can use the usual notation. For example, `url: $DEV_PROD_URL`.  If an environment variable is not set, the tool will fail. In addition, the system will also request for a set of required environment variables.
    - To learn about setting up different endpoint types such as HTTP/REST, HTTP/SOAP (with load balancing and failover), Dynamic and AWS Lambda, refer the section [Configuring Different Endpoint Types]({{base_path}}/learn/api-controller/advanced-topics/configuring-different-endpoint-types).
    -   To learn about setting up different endpoint security types such as Basic, Digest and OAuth 2.0, refer the section [Configuring Different Endpoint Security Types]({{base_path}}/learn/api-controller/advanced-topics/configuring-different-endpoint-security-types).
