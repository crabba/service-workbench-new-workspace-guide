===================================
Adding a Product in Service Catalog
===================================

.. _launch-constraint:

-----------------
Launch Constraint
-----------------

Every product launched by Service Catalog must have a role, or **Launch Constraint**, associated with it.  This role is assumed by the product and defines the permissions granted to that product.

---------------------------
Templated Launch Constraint
---------------------------

A standard Service Workbench deployment uses the same launch constraint for each product in the portfolio. This constraint is defined within the ``ServiceCatalogLaunchConstraintRole`` resource in the ``Resources`` section of the post-deployment template file.

File to edit:
  ``main/solution/post-deployment/config/infra/cloudformation.yml``

The packaged launch constraint is named ``xxx-LaunchConstraint`` and has policies:

    * ``CommonScProductLaunchPermissions``
    * ``EC2LaunchPermissions``
    * ``EMRLaunchPermissions``
    * ``SageMakerLaunchpermissions``

To add permissions for a new product, in the ``cloudformation.yml`` template file, create a new policy within ``ServiceCatalogLaunchConstraintRole``, following the pattern of the existing policies. The policy may include standard and custom configuration options, as shown above.

Example: Policy for allowing a workspace that creates a bucket. The bucket name stem is defined in a custom setting ``myCustomKey`` (see: :ref:`custom-options`).  Some actions have been removed for brevity.

.. highlight:: yaml

::

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
        
------------------------
Manual Launch Constraint
------------------------

It is possible in the AWS Console to create an individualized launch constraint for the new product.  To do this, the Service Catalog console, open the Portfolio and create a Constraint for the product.  Assign to the constraint a new Role with the following properties:

* Role name must match ``<stage>-<region>-<solution>-*LaunchConstraint`` ie ``demo-va-sw-myproduct-LaunchConstraint``
    * This requirement is enforced by policy ``backend-PolicyEnvMgmt`` in SC role ``EnvMgmt``
* Trust relationship: **Service Catalog** (``servicecatalog.amazonaws.com``)
* Has adequate permissions to launch the solution
