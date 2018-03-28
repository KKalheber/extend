#### Hosting the Editor in Your Web Site

Extend Editor can be hosted within your own web site and customized to provide your users with the most streamlined, built-in experience. Extend Editor can be added with just a few lines of script: 

```html
<!DOCTYPE html>
<html>
<head>
  <script src="https://cdn.auth0.com/auth0-extend/1/extend-editor.js"></script>
</head>
<body>
  <div id="extend-editor" style="height: 400px; width: 600px"></div>
  <script>
    ExtendEditor.create(document.getElementById('extend-editor'), {
      hostUrl: '{host_url}',
      webtaskContainer: '{webtask_container}',
      token: '{webtask_token}',
      webtaskName: 'webtask-sample',
      // If `webtask-sample` does not exist, Extend editor will create 
      // a webtask `webtask-sample` with a sample code.
      createIfNotExists: true 
    });
  </script>
</body>
</html>
```

The URL used above (https://cdn.auth0.com/auth0-extend/1/extend-editor.js) represents the **latest** version of the editor available in the 1.x version. You can specify a specific version by including the full URL. For example: https://cdn.auth0.com/auth0-extend/1.2.6/extend-editor.js. You can also request a specific patch level: https://cdn.auth0.com/auth0-extend/1.2/extend-editor.js. This version would automatically upgrade to 1.2.7 when and if it becomes available. You can keep track of the current version, and see fixes, at our [changelog](https://auth0.com/extend/editor/changelog).

In the typical situation, the Extend Editor would be presented to an authenticated user from the administration section of your web site, and the **{webtask_container}** and **{webtask_token}** would be specific to the tenant in your system they are managing. In a more general case, the **{host_url}**, **{webtask_container}**, and **{webtask_token}** are specific to the [selected isolation scope](#mapping-isolation-requirements-onto-webtask-tokens). 

[See Express handler that renders the page with Extend Editor](https://github.com/auth0/extend/blob/master/samples/zerocrm/routes/index.js#L22).  
[See how the Extend Editor is embedded in the settings page](https://github.com/auth0/extend/blob/master/samples/zerocrm/views/settings.ejs#L86).  

**NOTE** When experimenting with the code snippet above, you can use the same parameters as you use for running the [sample application](#sample-application). However, remember to never disclose your {master_webtask_token} to customers. 

The minimal configuration above will display the Extend Editor using all the default options:

![Default Extend Editor](https://cloud.githubusercontent.com/assets/302314/26526687/e34688aa-4358-11e7-8f17-9f3f222e3541.png)
