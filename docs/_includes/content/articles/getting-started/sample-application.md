## Sample Application

The ZeroCRM sample application at [auth0/extend](https://github.com/auth0/extend) is a working example of integrating Auth0 Extend into a hypothetical CRM system. As different integration aspects are discussed in this guide, you will see references to specific locations in the source code that demonstrate implementation details. 

![ZeroCRM](https://cloud.githubusercontent.com/assets/302314/24642275/29822bee-18bb-11e7-943d-2aa9d5b272b1.png)

To run the ZeroCRM application you will need the following configuration details: 

* **{host_url}**: The HTTPS URL for your Auth0 Extend instance. 
* **{master_webtask_token}**. This is the main webtask token you were provided with to manage your installation. 
* **{webtask_container}**. This is the main webtask container. 

If you are an existing Auth0 Extend customer, you were provided these as part of the onboarding process. If you are new to Auth0 Extend and want to try it out for free, you can get these parameters from the [Try Auth0 Extend](https://auth0.com/extend/try) page. 

To install the application, run the following: 

```bash
git clone https://github.com/auth0/extend.git
cd extend/samples/zerocrm
npm i
cat > .env <<EOF
MASTER_WEBTASK_TOKEN={master_webtask_token}
HOST_URL={host_url}
WEBTASK_CONTAINER={webtask_container}
ZEROCRM_TENANT={webtask_container}
EOF
```

Auth0 Extend has been designed to support code-based extensibility in multi-tenant systems. Normally the concept of a tenant is implied from the authentication context. For the sake of simplicity, the ZeroCRM sample application assumes the tenant name will be provided via an environment variable when you start the application. This can be provided by modifying the local .env file you create above and overriding `ZEROCRM_TENANT` with the tenant name. 

```bash
ZEROCRM_TENANT=tenant1
```

**NOTE** if you are using the [free trial of Auth0 Extend](https://auth0.com/extend/try), your account is **single-tenant** and cannot be used with arbitrary tenant names. In this case, please set the ZEROCRM_TENANT environment variable to the same value as {webtask_container}.