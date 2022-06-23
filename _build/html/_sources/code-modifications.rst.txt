------------------------------------
Service Workbench Code Modifications
------------------------------------

The Service Workbench code must be modified in several places to make the new product available.

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Create Service Catalog product
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This step creates a new product in the service catalog portfolio.  It refers to the template file previously created in :ref:`create-template-file`.

File to edit:
    ``addons/addon-base-raas/packages/base-raas-post-deployment/lib/steps/create-service-catalog-portfolio.js``

Add a block in ``productsToCreate``, describing the new product.

.. highlight:: javascript

::

    {
        filename: 'name of file in templates/service-catalog, without .cfn.yml extension',
        displayName: 'Unique display name',
        description: 'Your description here'
    }

.. _add-workspace-parameters:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Add Workspace Parameters to Configurations
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Any new parameters used by the CloudFormation template of your new product (see: :ref:`additional-parameters`) will require values when a new Service Workbench configuration is created for the new Workspace Type.  This step creates values to populate the configuration fields through a drop-down, in the same way as for the default workspace types.

**Define Parameters in Configuration Screen**

This step creates a new field in the Configuration screen for the new product, corresponding to each parameter in the template.

File to edit:
    ``addons/addon-base-ui/packages/base-ui/src/parts/helpers/fields/DropDown.js``

In the object ``fileToVariableMap``, add a key-value pair for each parameter defined in your product's template file.  The key will be the displayed field name, the value will be used as the key in the next step.  Both are in CamelCase.

Example:

.. highlight:: javascript

::

    const fieldToVariableMap = {
        // Default parameters
        EncryptionKeyArn: '${encryptionKeyArn}',
        Subnet: '${subnetId}',
        // My parameters
        MyNewParam: '{MyNewParamKey}'
    }

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Define Values for Configuration Values
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This step defines a default value for each key defined in the previous step :ref:`add-workspace-parameters`

File to edit:
    ``addons/addon-base-raas/packages/base-raas-services/lib/environment/service-catalog/environment-config-vars-service.js``

Add key-value pairs to the return value of ``resolveEnvConfigVars()``.  The keys must match the referred variable names in ``DropDown.js``:

``MyNewParamKey: 'someValue'``
