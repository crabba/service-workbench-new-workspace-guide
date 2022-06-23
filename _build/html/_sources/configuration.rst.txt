.. _custom-options:

----------------------------
Custom Configuration Options
----------------------------

Configuration key-value pairs may be added to the global Service workbench configuration file ``main/config/settings/<stage>.yml``.  Append any required values to this file prior to deployment or redeployment.

.. highlight:: yaml

::

    # My custom config value
    myCustomKey: 'someValue'

These values may be accessed from within CloudFormation templates throughout the deployment in the namespace ``self:custom.settings``:

.. highlight:: yaml

::

    someKey: !Sub "${self:custom.settings.myCustomKey}-foo"