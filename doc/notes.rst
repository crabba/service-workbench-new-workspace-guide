-----
Notes
-----

~~~~~~~~~~~~~~~~~~~~~~
Auto-Deployed Products
~~~~~~~~~~~~~~~~~~~~~~

Service Catalog will automatically deploy updates of products for which the user has not created a new version by uploading a template in the Service Catalog console.  To deploy an update, replace the template in ``templates/service-catalog``, and redeploy SWB.

~~~~~~~~~~~~~~~~~~~
Validating Products
~~~~~~~~~~~~~~~~~~~

The parameters of managed products are stored in a JSON object in the bucket ``env-type-configs/configs``.   Here, you can check that all the intended parameters are present by examining the ``params`` array.

.. highlight:: javascript

::

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
