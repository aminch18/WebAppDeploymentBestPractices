# WebApp Deployment Best Practices

## Deployment Components

### Deployment Source
A deployment source is the location of your application code. For production apps, the deployment source is usually a repository hosted by version control software such as GitHub, BitBucket, or Azure Repos. 

### Build Pipeline

Once you decide on a deployment source, your next step is to choose a build pipeline. A build pipeline reads your source code from the deployment source and executes a series of steps (such as compiling code, minifying HTML and JavaScript, running tests, and packaging components) to get the application in a runnable state.

### Deployment Mechanism

The deployment mechanism is the action used to put your built application into the wwwroot directory of your web app.

App Service supports the following deployment mechanisms:

- Web Deploy
- Kudu REST APIs
- Container Registry
- Zip Deploy
- Run From Package
- War Deploy ( Java Apps )

## Different types of deployments and which one to use

### Zip Deploy

Expects a .zip deployment package and deploys the file contents to the wwwroot folder of the App Service or Function App in Azure. This option overwrites all existing contents in the wwwroot folder. For more information, see Zip deployment for Azure Functions.

### Run From Package

Expects the same deployment package as Zip Deploy. However, instead of deploying files to the wwwroot folder, the entire package is mounted by the Functions runtime and files in the wwwroot folder become read-only. For more information, see Run your Azure Functions from a package file.

### Web Deploy (MSDeploy)

Web Deploy (msdeploy.exe) can be used to deploy a **Web App on Windows or a Function App to the Azure App Service using a Windows agent**. Web Deploy is feature-rich and offers options such as:

- Rename locked files: Rename any file that is still in use by the web server by enabling the msdeploy flag MSDEPLOY_RENAME_LOCKED_FILES=1 in the Azure App Service settings. This option, if set, enables msdeploy to rename files that are locked during app deployment.

- Remove additional files at destination: Deletes files in the Azure App Service that have no matching files in the App Service artifact package or folder being deployed.

- Exclude files from the App_Data folder: Prevent files in the App_Data folder (in the artifact package/folder being deployed) being deployed to the Azure App Service

- Additional Web Deploy arguments: Arguments that will be applied when deploying the Azure App Service. Example: -disableLink:AppPoolExtension -disableLink:ContentExtension. For more examples of Web Deploy operation settings, see Web Deploy Operation Settings.

### Kudu REST APIs

Kudu REST APIs work on both Windows and Linux automation agents when the target is a Web App on Windows, Web App on Linux (built-in source), or Function App. The task uses Kudu to copy files to the Azure App service.

### Container Registry

Works on both Windows and Linux automation agents when the target is a Web App for Containers. The task updates the app by setting the appropriate container registry, repository, image name, and tag information. You can also use the task to pass a startup command for the container image.

## Roll Out strategies with 0 down time

### Deployment slots

Whenever possible, use deployment slots when deploying a new production build. When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment, validate your changes, and do smoke tests. When you are ready, you can swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.

1. Apply the following settings from the target slot (for example, the production slot) to all instances of the source slot:

2. Wait for every instance in the source slot to complete its restart. If any instance fails to restart, the swap operation reverts all changes to the source slot and stops the operation.

3. If local cache is enabled, trigger local cache initialization by making an HTTP request to the application root ("/") on each instance of the source slot. Wait until each instance returns any HTTP response. Local cache initialization causes another restart on each instance.

4. If auto swap is enabled with custom warm-up, trigger Application Initiation by making an HTTP request to the application root ("/") on each instance of the source slot.

5. If applicationInitialization isn't specified, trigger an HTTP request to the application root of the source slot on each instance.

6. If an instance returns any HTTP response, it's considered to be warmed up.

7. If all instances on the source slot are warmed up successfully, swap the two slots by switching the routing rules for the two slots. After this step, the target slot (for example, the production slot) has the app that's previously warmed up in the source slot.

8. Now that the source slot has the pre-swap app previously in the target slot, perform the same operation by applying all settings and restarting the instances.

### Preform smoke web tests during deploy to ensure your api/web is responding

Smoke web tests consist of simple http requests to make sure everything is working (or at least responding).

https://marketplace.visualstudio.com/items?itemName=miguelcruz.vsts-smoke-web-test-task

## Deployment Configuration

### Always On

Keeps the app loaded even when there's no traffic. It's required for continuous WebJobs or for WebJobs that are triggered using a CRON expression.

### ARR affinity

In a multi-instance deployment, ensure that the client is routed to the same instance for the life of the session. You can set this option to Off for stateless applications.

There are situations where in keeping the affinity is not desired. For example, if you are getting way too many requests from a single user and the requests going to the same web server instance can overload it. If maintaining session affinity is not important and you want better load balancing , it is recommended to disable session affinity cookie. 

### HTTP version 2.0

The primary goals for HTTP/2 are to reduce latency by enabling full request and response multiplexing, minimize protocol overhead via efficient compression of HTTP header fields, and add support for request prioritization and server push.

- HTTP/2 is binary
- Fully multiplexed, instead of ordered and blocking
- Ability to use one connection for parallelism
- Has one TCP/IP connection
- Uses header compression to reduce overhead

### Local Cache

Local Cache is like a temporary area where the App service can store all the content related to the App Service. Instead of referring a shared location, the App Service stores all its content in each provisioned VM. So, if the App Service is being configured in 2 VMs, all the content would be copied in both the VMs. So, obviously reading the content from the local copy is better than reading them from a shared location and as a result, there would a bit of improvement of the performance when we enable the “App Service Local Cache” feature.

Local cache is not supported in function apps or containerized App Service apps, such as in Windows Containers or in App Service on Linux.

All deployments done by all different methods (FTP, Web Deploy etc.) are pointed to the Shared Content area. So, in order to get these changes reflected to the local cache, the App Service needs to be restarted in order to get the new changes reflected.

## Differences between tiers

- Shared compute: Free and Shared, the two base tiers, runs an app on the same Azure VM as other App Service apps, including apps of other customers. These tiers allocate CPU quotas to each app that runs on the shared resources, and the resources cannot scale out.

- Dedicated compute: The Basic, Standard, Premium, PremiumV2, and PremiumV3 tiers run apps on dedicated Azure VMs. Only apps in the same App Service plan share the same compute resources. The higher the tier, the more VM instances are available to you for scale-out.

- Isolated: This tier runs dedicated Azure VMs on dedicated Azure Virtual Networks. It provides network isolation on top of compute isolation to your apps. It provides the maximum scale-out capabilities.

https://azure.microsoft.com/en-us/pricing/details/

### Scale up (Vertical scale)

Get more CPU, memory, disk space, and extra features like dedicated virtual machines (VMs), custom domains and certificates, staging slots, autoscaling, and more. You scale up by changing the pricing tier of the App Service plan that your app belongs to.

### Scale up (Horizontal scale)

Increase the number of VM instances that run your app. You can scale out to as many as 30 instances, depending on your pricing tier. App Service Environments in Isolated tier further increases your scale-out count to 100 instances. 

### Autoscale

Autoscale only scales horizontally, which is an increase ("out") or decrease ("in") in the number of VM instances. Horizontal is more flexible in a cloud situation as it allows you to run potentially thousands of VMs to handle load.

Estimation during a scale-in is intended to avoid "flapping" situations, where scale-in and scale-out actions continually go back and forth. Keep this behavior in mind when you choose the same thresholds for scale-out and in.