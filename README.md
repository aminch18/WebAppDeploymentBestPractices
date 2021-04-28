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
 
### Deployment slots

Whenever possible, use deployment slots when deploying a new production build. When using a Standard App Service Plan tier or better, you can deploy your app to a staging environment, validate your changes, and do smoke tests. When you are ready, you can swap your staging and production slots. The swap operation warms up the necessary worker instances to match your production scale, thus eliminating downtime.

### Local Cache

Azure App Service content is stored on Azure Storage and is surfaced up in a durable manner as a content share. However, some apps just need a high-performance, read-only content store that they can run with high availability. These apps can benefit from using local cache. Local cache is not recommended for content management sites such as WordPress.

Local cache is not supported in function apps or containerized App Service apps, such as in Windows Containers or in App Service on Linux.

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

When you deploy your web app, you can use a separate deployment slot instead of the default production slot when you're running in the Standard, Premium, or Isolated App Service plan tier. Deployment slots are live apps with their own host names. App content and configurations elements can be swapped between two deployment slots, including the production slot.

1. Apply the following settings from the target slot (for example, the production slot) to all instances of the source slot:

2. Wait for every instance in the source slot to complete its restart. If any instance fails to restart, the swap operation reverts all changes to the source slot and stops the operation.

3. If local cache is enabled, trigger local cache initialization by making an HTTP request to the application root ("/") on each instance of the source slot. Wait until each instance returns any HTTP response. Local cache initialization causes another restart on each instance.

4. If auto swap is enabled with custom warm-up, trigger Application Initiation by making an HTTP request to the application root ("/") on each instance of the source slot.

5. If applicationInitialization isn't specified, trigger an HTTP request to the application root of the source slot on each instance.

6. If an instance returns any HTTP response, it's considered to be warmed up.

7. If all instances on the source slot are warmed up successfully, swap the two slots by switching the routing rules for the two slots. After this step, the target slot (for example, the production slot) has the app that's previously warmed up in the source slot.

8. Now that the source slot has the pre-swap app previously in the target slot, perform the same operation by applying all settings and restarting the instances.

## Preform smoke web tests during deploy to ensure your api/web is responding