============
Introduction
============

The following steps are required to define a workspace type from an existing CloudFormation template of a solution, and integrate the template into Service Workbench, along with creating the necessary resources such as IAM roles.

The code examples were taken from a workspace type to create an S3 bucket.  Similar patterns may be followed for other new workspace types.

--------
Overview
--------

- Each new Workspace Type in Service workbench ultimately comes from a CloudFormation template defining a solution.
- The CloudFormation template first is imported into Service Catalog as a new Product. Each Product in Service Catalog will correspond to a Workspace Type in Service Workbench.
- Workspaces in Service Workbench are created from a Configuration of a Workspace Type. Each Configuration contains some fixed values (such as an AMI ID, or instance type) and some variables. These values (fixed and variable) are provided to the Workspace when it is launched, to provide customization of the template being launched.
- Service Workbench requires some code modifications for each new added product (workspace type).  This is best enabled by creating in GitHub a fork (copy) of the original Service Workbench repository, modifying the copy in this repository, and then deploying this code.
- Service Workbench includes a CI-CD pipeline which can automatically redeploy Service Workbench each time a commit is made to a specified branch of the forked repo.
- The forked repo may be synchronized with the original repo, keeping your copy up to date with any advancements made in the original code.  Any conflicts (the same line edited in both repos) will be identified by Git and presented for manual resolution before deployment.
