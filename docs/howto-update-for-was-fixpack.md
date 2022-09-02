# How to update the tWAS base single server on Azure VM solution for next tWAS fixpack

Please follow sections below in order to update the solution for next tWAS base fixpack.

## Updating the image

1. Which file to update for WAS version?
   * For `twas-base` image, update the following properties in file [`virtualimage.properties`](https://github.com/WASdev/azure.websphere-traditional.image/blob/main/twas-base/src/main/scripts/virtualimage.properties#L14-L15), e.g.:

     ```bash
     WAS_BASE_TRADITIONAL=com.ibm.websphere.BASE.v90
     IBM_JAVA_SDK=com.ibm.java.jdk.v8
     ```

     Note: only the major version should be specified, the minor version should not be hard-coded as the Installation Manager will intelligently install the latest available minor version.

1. When to update the images?
- For new tWAS fixpack, try to update the image soon after the fixpack GA but no longer than one week after the GA.
- Images may also need to updated to fix a critical WebSphere or OS fixes.

1. How to run CI/CD?
   * Go to [Actions](https://github.com/WASdev/azure.websphere-traditional.image/actions) > Click `twas-base CICD` > Click to expand `Run workflow` > Click `Run workflow` > Refresh the page

1. How to test the image, what testcases to run?
   * The CI/CD contains tests to verify the entitlement check and tWAS installation, so basically it's good to go without manual tests.
   * However, if CI/CD failed, please look at error messages from the CI/CD logs, and [access the source VM](https://github.com/WASdev/azure.websphere-traditional.image/blob/main/docs/howto-access-source-vm.md) for troubleshooting if necessary.

1. How to publish the image **as a hidden image** in marketplace and who can do it?
   1. Wait until the CI/CD workflow for `twas-base CICD` successfully completes > Click to open details of the workflow run > Scroll to the bottom of the page > Click `sasurl` to download the zip file `sasurl.zip` > Unzip and open file `sas-url.txt` > Find values for `osDiskSasUrl` and `dataDiskSasUrl`;
   1. Sign into [Microsoft Partner Center](https://partner.microsoft.com/dashboard/commercial-marketplace/overview):
      * Select the Directory `IBM-Alliance-Microsoft Partner Network-Global-Tenant`
      * Expand `Build solutions` and choose `Publish your solution`.  
      * Click to open the offer for `2022-01-06-twas-single-server-base-image`
      * Click `Plan overview` the click to open the plan 
      * **IMPORTANT** Click `Pricing and availability` to verify the plan is hidden from the marketplace
         * Ensure the `Hide plan` checkbox is checked
      * Click `Technical configuration` 
      * Click `+ Add VM image` > Specify a new value for `Disk version`, following the convention \<major version\>.YYYYMMDD, e.g. 9.0.20210929 and write it down (We deliberately do not specify the minor verson because the pipeline gets the latest at the time it is run). 
      * Select `SAS URI` > Copy and paste value of `osDiskSasUrl` for `twas-base` to the textbox `SAS URI` 
      * Click `+ Add data disk (max 16)` > Select `Data disk 0` > Copy and paste value of `dataDiskSasUrl` for `twas-base` to the textbox `Data disk VHD link`
      * Scroll to the bottom of the page and click `Save VM image`
      * Click `Save draft`
      * Click `Review and publish`
      * In the "Notes for certification" section enter the twas-base CICD URL
      * Click `Publish`;
      * Wait for few hours to a day, keep refreshing the page until "Go Live" button appears
      * Click on "Go Live" and wait again (for few hours) for the image to be published.
      * **Note:** After the image is successfully published and available, please [clean up the storage account with VHD files](https://github.com/WASdev/azure.websphere-traditional.image/blob/main/docs/howto-cleanup-after-image-published.md) for reducing Azure cost.
      * Now proceed to [Updating and publishing the solution code](#updating-and-publishing-the-solution-code) steps

   Note: Currently Graham Charters has privilege to update the image in marketplace, contact him for more information.

1. Do we need to update the solution every time we do the image update?
   * Yes. That's because image version of [`twas-base`](https://github.com/WASdev/azure.websphere-traditional.singleserver/blob/main/src/main/bicep/config.json#L13) is explicitely referenced in the tWAS base single server solution. Make sure correct image version is specified in the `config.json` of the solution code.

## Updating and publishing the solution code

Note: **Wait for images to be published before proceeding with this step.** The steps included in this section are also applied to release new features / bug fixes which have no changes to the images.

1. How to update the version of the solution?
   * Increase the [version number](https://github.com/WASdev/azure.websphere-traditional.singleserver/blob/main/pom.xml#L22) which is specified in the `pom.xml`
   * Also update the [`twasImageVersion`](https://github.com/WASdev/azure.websphere-traditional.singleserver/blob/main/src/main/bicep/config.json#L13) in the `config.json` (obtained from publish step)
   * Get the PR merged

1. How to run CI/CD?
   * Go to [Actions](https://github.com/WASdev/azure.websphere-traditional.singleserver/actions) > Click `integration-test` > Click to expand `Run workflow` > Click `Run workflow` > Refresh the page

1. How to publish the solution in marketplace and who can do it? (**Note: Make sure the image is published before publishing the solution**)
   1. Wait until the CI/CD workflow for `integration-test` successfully completes 
       * Click to open details of the workflow run > Scroll to the bottom of the page
       * Click `azure.websphere-traditional.singleserver-<version>-arm-assembly` to download the zip file `azure.websphere-traditional.singleserver-<version>-arm-assembly.zip`;
   3. Sign into [Microsoft Partner Center](https://partner.microsoft.com/dashboard/commercial-marketplace/overview)
       * Click to open the offer for the solution (likely `2022-01-07-twas-base-single-server`) > Click `Plan overview`
       * Click to open the plan > Click `Technical configuration`
       * Specify the increased version number for `Version` (note, the version is in the zip file name)
       * Click `Remove` to remove the previous package file
       * Click `browse your file(s)` to upload the downloaded zip package generated by the CI/CD pipeline before
       * Scroll to the bottom of the page
       * Click `Save draft`
       * Click `Review and publish`
       * In the "Notes for certification" section enter the `integration-test` URL
       * Click `Publish`
       * Wait until solution offer is in `Publisher signoff` (aka "preview") stage and "Go Live" button appears(it could take few hours)
       * Before clicking "Go Live" use the preview link to test the solution
       * Run test cases defined in [twas-solution-test-cases.md](twas-solution-test-cases.md). Note: use "preview link" for each test case.
       * Click "Go Live"
       * Wait for remaining steps to complete (may take couple of days)
       * Make sure to delete your test deployments
       * Once the solution is in "Publish" stage, new version is publicly available
       * To verify the version number, launch the solution in Azure portal and hover over "Issue tracker" and it should display the version number. For example, https://aka.ms/azure-twas-singleserver-issues?version=**1.0.1**

   Note: Currently Graham Charters has privilege to update the solution in marketplace, contact him for more information.

1. Create a [release](https://github.com/WASdev/azure.websphere-traditional.singleserver/releases) for this GA code and tag with the pom.xml version number.


## What needs to be cleaned up from test env and how to clean them up?

Azure marketplace is responsible for managing different stages during the offer publishing, just follow its process to make it Go-Live and no additional clean-ups are needed.

## Do we delete/archive previous version of the solution?

Previous versions of the solution are archived. You can find/download them from "Offer > Plan overview > Technical configuration > Previously published packages".

## Create a release and a branch with the GA code (for image and singleserver repo)

Probably creating a release/tag for each GA code is good enough.


## Troubleshooting
### twas-base CICD's Deploy VM stage fails with "Can not perform requested operation on nested resource"
```
ERROR: ***"status":"Failed","error":***"code":"DeploymentFailed","message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/DeployOperations for usage details.","details":[***"code":"NotFound","message":"***\r\n  \"error\": ***\r\n    \"code\": \"ParentResourceNotFound\",\r\n    \"message\": \"Can not perform requested operation on nested resource. Parent resource 'evaluation297509892811' not found.\"\r\n  ***\r\n***"***]***
Error: Process completed with exit code 1.
```

This failure is caused by the issue that vm extension can't find the VM where it's executed. I guess it's an intermittent Azure issue as I can't reproduce the similar issue.
