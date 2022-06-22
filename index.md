# Adding a New Product to Service Workbench

## Introduction
* The following steps are required to define a workspace type from an existing CloudFormation template of a solution, and integrate the template into Service Workbench, along with creating the necessary resources such as IAM roles.
* The code examples were taken from a workspace type to create an S3 bucket.  Similar patterns may be followed for other new workspace types.

## Overview
- Each new Workspace Type in Service workbench ultimately comes from a CloudFormation template defining a solution.
- The CloudFormation template first is imported into Service Catalog as a new Product. Each Product in Service Catalog will correspond to a Workspace Type in Service Workbench.
- Workspaces in Service Workbench are created from a Configuration of a Workspace Type. Each Configuration contains some fixed values (such as an AMI ID, or instance type) and some variables. These values (fixed and variable) are provided to the Workspace when it is launched, to provide customization of the template being launched.
- Service Workbench requires some code modifications for each new added product (workspace type).  This is best enabled by creating in GitHub a fork (copy) of the original Service Workbench repository, modifying the copy in this repository, and then deploying this code.
- Service Workbench includes a CI-CD pipeline which can automatically redeploy Service Workbench each time a commit is made to a specified branch of the forked repo.
- The forked repo may be synchronized with the original repo, keeping your copy up to date with any advancements made in the original code.  Any conflicts (the same line edited in both repos) will be identified by Git and presented for manual resolution before deployment.

## Suggested Workflow
### Code Repositories
  - In GitHub, create a fork of the Service Workbench repo [awslabs/service-workbench-on-aws](https://github.com/awslabs/service-workbench-on-aws).  Your fork may be either public or private. 
  - Clone your GitHub repo `mylogin/service-workbench-on-aws` to your local file system.  This will create a remote repo named `origin`, pointing to your GitHub repo.
      - `git clone mygithub git@github.com:mylogin/service-workbench-on-aws.git`
  - The main branch of the repo is `mainline`. Create another branch of the repo for your work.
      - `git checkout -b my-dev`
  - Perform your local edits and commit to your local dev branch, then when complete, merge with your main branch.
      - `git checkout mainline`
      - `git merge my-dev`
  - Sync option 1: Sync locally
      - You may define the original (AWS) repo as a second remote repository (conventionally named `upstream`), and sync it to your local repo after you have merged your edits to your `mainline` branch.
      - `git remote add upstream https://github.com/awslabs/amazon-service-workbench-on-aws.git`
      - `git pull upstream mainline`
  - For either sync option, push your local repo to your GitHub repo
      - `git push origin mainline`
  - Sync Option 2: Sync on GitHub
      - With your code pushed to your repo in GitHub, sync the AWS code into your code by clicking **Fetch upstream** -> **Fetch and merge**
  - Your GitHub repo now holds the Service Workbench source code, including your edits, and up to date with the upstream repo.
  ### CI-CD Pipeline
  - Optionally, you may enable the CI-CD pipeline to automatically redeploy Service Workbench upon each new commit to a designated branch of your GitHub repo.
  - The CI-CD pipeline will not deploy configuration files, since all configuration files are excluded from the git repo by the top-level `.gitignore` file.  Perform the initial Service Workbench installation from a deployment instance, before performing subsequent redeploys using the CI-CD pipeline.  Configuration changes are achieved by uploading edited configuration files to the relevant location in the `artifacts` bucket: e.g.: the main config file goes in the `settings` folder.
  - Read the files `docs/docs/best_practices/cicd.md` and `main/cicd/README.md` for an explanation of the two stages `cicd-pipeline` and `cicd-source`.   Only `cicd-pipeline` is necessary to deploy Service Workbench directly from your GitHub repo; `cicd-code` creates another repository in CodeCommit.
  - Create a config file in `main/cicd/cicd-pipeline/config/settings/` by copying `example-github.yml` and naming it `<stage>.yml`. 
  - Edit the config file with the details of your GitHub repo.
  - The **awsProfile** value refers to the profile on the build instance used by CodeBuild; use the value `awsProfile: default`.  
      - **IMPORTANT**: If you define the `awsProfile` value in the `cicd-pipeline` config file, you must **not** define the `awsProfile` value in the Service Workbench main configuration file (`main/config/settings/<stage.yml`).  Comment out this line.
  - Once configured, commits to the GitHub repo specified above will trigger a CodePipeline pipeline which will retrieve the code from GitHub, and deploy Service Workbench using a CodeBuild build project.  You will need to provide access to your GitHub repo using the **Connect to GitHub** mechanism within CodePipeline.

## Creating a Service Workbench-compatible template file
* This section involves adding a new CloudFormation template file to the directory holding the templates for the existing workspace types.  The directory is:
    * `addons/addon-base-raas/packages/base-raas-cfn-templates/src/templates/service-catalog`
* **Installation**
    * Add the modified CloudFormation template file to this directory, alongside the existing template files (`ec2-linux-instance.cfn.yml` etc). Commit the new file to the Git repository.
    * Every template file in this directory will be added to Service Catalog as a product, upon deployment of Service Workbench. If the template file is subsequently modified, a new version of the product will be created in Service Catalog upon redeployment of Service Workbench.  Each new version of a product in Service Catalog will create a new Workspace Type within Service Workbench, which must then be imported and have a Configuration created for it.
    * Every product in Service Catalog will have the same launch constraint (see below) assigned to it.  This launch constraint will probably need to be updated with the permissions needed for the new product.
* **Required Parameters**
    * The following Parameters must exist in the template:
        * `IsAppStreamEnabled`
        * `EgressStoreIamPolicyDocument`
        * `SolutionNamespace`
    * Example:
    * ```yaml
      IsAppStreamEnabled:
        Type: String
        AllowedValues: [true, false]
        Description: Is AppStream enabled for this workspace
        Default: "false"
      EgressStoreIamPolicyDocument:
        Type: String
        Description: IAM policy for launched workstation to access egress store
      SolutionNamespace:
        Type: String
        Description: Namespace value provided when onboarding Member account
      ```
* **Additional Parameters**
    * You may use any parameters in your product template. Later we will define in Service Workbench a means for entering these parameters when creating a configuration for this product.
    * Example:
    * ```yaml
      MyNewParam:
        Type: String
        Default: foo
        AllowedValues:
        * foo
        * bar
        Description: Gathers my new parameter.  Valid values are 'foo' and 'bar' (default: foo)```
* **Outputs**
    * Add the following Outputs to your CloudFormation template.  Follow the example in e.g.: `ec2-linux-instance.cfn.yml`
        * `MetaConnection1Name`: Name for connection
        * `MetaConnection1Type`:  Type of workspace
        * `MetaConnection1Info`: Supplementary information
        * `MetaConnection1Url`: URL for connection to the new workspace
        * `MetaConnection1Scheme`: Protocol for connection
    * Example:
    * ```yaml
      Outputs:
        MetaConnection1Name:
          Description: Name for bucket
          Value: !Ref Bucket
        MetaConnection1Url:
          Description: URL for bucket
          Value: !Join ['', ['https://s3.console.aws.amazon.com/s3/buckets/', !Ref Bucket, "?region=", !Ref AWS::Region]]
        MetaConnection1Scheme: 
          Description: Protocol for bucket
          Value: https
        MetaConnection1Type:
          Description: Type for bucket
          Value: S3
        MetaConnection1Info: 
          Description: Info for bucket
          Value: foo```

## Custom Configuration Options
* Configuration key-value pairs may be added to the global Service workbench configuration file `main/config/settings/<stage>.yml`.  Append any required values to this file prior to deployment or redeployment.
  * ```yaml
    # My custom config value
    myCustomKey: 'someValue'
    ```
  * These values may be accessed from within CloudFormation templates throughout the deployment in the namespace `self:custom.settings`:
  * ```javascript
    someKey: !Sub "${self:custom.settings.myCustomKey}-foo"
    ```

## Adding a Product in Service Catalog

### Templated Launch Constraint

  * A standard Service Workbench deployment uses the same launch constraint for each product in the portfolio. This constraint is defined within the `ServiceCatalogLaunchConstraintRole` resource in the `Resources` section of the template `main/solution/post-deployment/config/infra/cloudformation.yml`
  * The packaged launch constraint is named `xxx-LaunchConstraint` and has policies:
      * `CommonScProductLaunchPermissions`
      * `EC2LaunchPermissions`
      * `EMRLaunchPermissions`
      * `SageMakerLaunchpermissions`
  * To add permissions for a new product, in `cloudformation.yml` create a new policy within `ServiceCatalogLaunchConstraintRole`, following the pattern of the existing policies. The policy may include standard and custom configuration options, as shown above.
  *  Example: Policy for allowing a workspace that creates a bucket. The bucket name stem is defined in a custom setting `myCustomKey`.  Some actions have been removed for brevity.
      * ```yaml
        # Allow create-bucket workspace
        - PolicyName: S3LaunchPermissions
          PolicyDocument:
            Version: "2012-10-17"
            Statement:
        - Effect: Allow
              Action:
            - s3:GetObject
              Resource:
            - !Sub arn:aws:s3:::${self:custom.settings.myCustomKey}*
        ```

### Manual Launch Constraint
  * It is possible in the AWS Console to create an individualized launch constraint for the new product. 
  * In the Service Catalog console, open the Portfolio and create a Constraint for the product.  Assign to the constraint a new Role with the following properties:
      * Role name must match `<stage>-<region>-<solution>-*LaunchConstraint` ie `demo-va-sw-myproduct-LaunchConstraint`
          * This requirement is enforced by policy `backend-PolicyEnvMgmt` in SC role `EnvMgmt`
      * Trust relationship: **Service Catalog** (`servicecatalog.amazonaws.com`)
      * Has adequate permissions to launch the solution

## Service Workbench Code Modifications

* The Service Workbench code must be modified in several places to make the new product available.  

### Create Service Catalog product

  * This step creates a new product in the service catalog portfolio.
  * It refers to the template file created in **Creating a Service Workbench-compatible template file**, above
  * File to edit:
      * `addons/addon-base-raas/packages/base-raas-post-deployment/lib/steps/create-service-catalog-portfolio.js`
  * Add block in `productsToCreate` describing new product
  * ```javascript
    {
        filename: ‘name of file in templates/service-catalog, without .cfn.yml extension’,
        displayName: ‘Unique display name’,
        description: `Your description here`,
    },
    ```

### Add Workspace Parameters to Configurations

  * Any new parameters used by the CloudFormation template of your new product will require values when a new Service Workbench configuration is created for the new Workspace Type.  This step creates values to populate the configuration fields through a drop-down, in the same way as for the default workspace types.
  * **Define Parameters in Configuration Screen**
    * This step creates a new field in the Configuration screen for the new product, corresponding to each parameter in the template.
    * File to edit:
        * `addons/addon-base-ui/packages/base-ui/src/parts/helpers/fields/DropDown.js`
    * In the object `fileToVariableMap`, add a key-value pair for each parameter defined in your product's template file.  The key will be the displayed field name, the value will be used as the key in the next step.  Both are in CamelCase,
    * Example:
    * ```javascript
      const fieldToVariableMap = {
        // Default parameters
        EncryptionKeyArn: '${encryptionKeyArn}',
        Subnet: '${subnetId}',
        // My parameters
        MyNewParam: '{MyNewParamKey}'
      }
      ```
  * **Define Values for Configuration Values**
    * This step defines a default value for each key defined in the previous step.
    * File to edit:
        * `addons/addon-base-raas/packages/base-raas-services/lib/environment/service-catalog/environment-config-vars-service.js`
    * Add key-value pairs to the return value of `resolveEnvConfigVars()`
    * Keys must match the referred variable names in `DropDown.js`:
        * `MyNewParamKey: 'someValue'`



## Notes

### Auto-Deployed Products
  * Service Catalog will automatically deploy updates of products for which the user has not created a new version by uploading a template in the Service Catalog console.  To deploy an update, replace the template in `templates/service-catalog`, and redeploy SWB.

### Validating Products
  * The parameters of managed products are stored in a JSON object in the bucket `env-type-configs/configs`.   Here, you can check that all the intended parameters are present by examining the `params` array.
```javascript
{
    "items": [{
        "id": "foo",
        "name": "foo",
        "desc": "foo",
        "estimatedCostInfo": "foo",
        "allowRoleIds": ["admin", "researcher"],
        "denyRoleIds": [],
        "params": [{
            "key": "param1",
            "value": "${MetaVariable}"
        }, {
            "key": "param2",
            "value": "some-absolute-value"
        }],
        "tags": [],
        "createdBy": "u-MSaGSVlb_oE-pS9SUfPM6",
        "updatedBy": "u-MSaGSVlb_oE-pS9SUfPM6",
        "createdAt": "2021-09-17T00:12:44.029Z",
        "updatedAt": "2021-09-17T00:12:44.029Z"
    }]
}
```
