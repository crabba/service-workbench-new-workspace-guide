.. _create-template-file:

=====================================================
Creating a Service Workbench-compatible template file
=====================================================

This section involves adding a new CloudFormation template file to the directory holding the templates for the existing workspace types.  The directory is:

``addons/addon-base-raas/packages/base-raas-cfn-templates/src/templates/service-catalog``

------------
Installation
------------

Add the modified CloudFormation template file to this directory, alongside the existing template files (``ec2-linux-instance.cfn.yml`` etc). Commit the new file to the Git repository.

Every template file in this directory will be added to Service Catalog as a product, upon deployment of Service Workbench. If the template file is subsequently modified, a new version of the product will be created in Service Catalog upon redeployment of Service Workbench.  Each new version of a product in Service Catalog will create a new Workspace Type within Service Workbench, which must then be imported and have a Configuration created for it.

Every product in Service Catalog will have the same :ref:`launch-constraint` assigned to it.  This launch constraint will probably need to be updated with the permissions needed for the new product.

-------------------
Required Parameters
-------------------

The following Parameters must exist in the template:
    * ``IsAppStreamEnabled``
    * ``EgressStoreIamPolicyDocument``
    * ``SolutionNamespace``

Example:

.. highlight:: yaml

::

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

.. _additional-parameters:

---------------------
Additional Parameters
---------------------

You may use any parameters in your product template. Later we will define in Service Workbench a means for entering these parameters when creating a configuration for this product (see :ref:`add-workspace-parameters`)

Example:

::

    MyNewParam:
        Type: String
        Default: foo
        AllowedValues:
            - foo
            - bar
        Description: Gathers my new parameter.  Valid values are 'foo' and 'bar' (default: foo)

-------
Outputs
-------

Add the following Outputs to your CloudFormation template.  Follow the example in e.g.: ``ec2-linux-instance.cfn.yml``

* ``MetaConnection1Name``: Name for connection
* ``MetaConnection1Type``:  Type of workspace
* ``MetaConnection1Info``: Supplementary information
* ``MetaConnection1Url``: URL for connection to the new workspace
* ``MetaConnection1Scheme``: Protocol for connection

Example:

::

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
        Value: foo
