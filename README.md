# WebApp Deployment Best Practices
 
## Different types of deployments and which one to use

### Zip deploy

All other deployment methods in App Service have something in common: your files are deployed to D:\home\site\wwwroot in your app (or /home/site/wwwroot for Linux apps). Since the same directory is used by your app at runtime, it's possible for deployment to fail because of file lock conflicts, and for the app to behave unpredictably because some of the files are not yet updated.

In contrast, when you run directly from a package, the files in the package are not copied to the wwwroot directory. Instead, the ZIP package itself gets mounted directly as the read-only wwwroot directory. There are several benefits to running directly from a package:

- Eliminates file lock conflicts between deployment and runtime.
- Ensures only full-deployed apps are running at any time.
- Can be deployed to a production app (with restart).
- Improves the performance of Azure Resource Manager deployments.
- May reduce cold-start times, particularly for JavaScript functions with large npm package trees.

When using ZipDeploy, files will only be copied if their timestamps don't match what is already deployed.

To consider:

- Running directly from a package makes wwwroot read-only. Your app will receive an error if it tries to write files to this directory.
- The ZIP file can be at most 1GB
- This feature is not compatible with local cache.
- For improved cold-start performance, use the local Zip option (WEBSITE_RUN_FROM_PACKAGE=1).


### Docker container


### Web Deploy (MSDeploy)


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