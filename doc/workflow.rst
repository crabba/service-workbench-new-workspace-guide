==================
Suggested Workflow
==================

-----------------
Code Repositories
-----------------

In GitHub, create a fork of the Service Workbench repo `awslabs/service-workbench-on-aws <https://github.com/awslabs/service-workbench-on-aws>`_.  Your fork may be either public or private. 

~~~~~~~~~~~~~~~~~~~~~~~~~
Create a Local Repository
~~~~~~~~~~~~~~~~~~~~~~~~~

Clone your GitHub repo ``mylogin/service-workbench-on-aws`` to your local file system.  This will create a remote repo named ``origin``, pointing to your GitHub repo.

``git clone mygithub git@github.com:mylogin/service-workbench-on-aws.git``

~~~~~~~~~~~~~~~
Branch and Edit
~~~~~~~~~~~~~~~

The main branch of the repo is ``mainline``. Create another branch of the repo for your work.

``git checkout -b my-dev``

Perform your local edits and commit to your local dev branch, then when complete, merge with your main branch.

``git checkout mainline``
``git merge my-dev``

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Sync with Upstream, Push to GitHub
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Your local codebase contains your edits, and before deployment must be merged with any changes made in the main AWS code repository since your last sync or pull from that repo. This synchronization may occur either locally (before pushing to your GitHub repo), or on GitHub, after your edited code has been pushed to your repo.

**Sync option 1: Sync locally**

You may define the original (AWS) repo as a second remote repository (conventionally named ``upstream``), and sync it to your local repo after you have merged your edits to your ``mainline`` branch.  This is performed before pushing the code to the GitHub repo.

``git remote add upstream https://github.com/awslabs/amazon-service-workbench-on-aws.git``

``git pull upstream mainline``

**Push to GitHub**

For either sync option, push your local repo to your GitHub repo.  

``git push origin mainline``

**Sync Option 2: Sync on GitHub**

With your code pushed to your repo in GitHub, sync the AWS code into your code by clicking **Fetch upstream** -> **Fetch and merge**. Your GitHub repo now holds the Service Workbench source code, including your edits, and up to date with the upstream repo.

--------------
CI-CD Pipeline
--------------

Optionally, you may enable the CI-CD pipeline to automatically redeploy Service Workbench upon each new commit to a designated branch of your GitHub repo.

The CI-CD pipeline will not deploy configuration files, since all configuration files are excluded from the git repo by the top-level ``.gitignore`` file.  Perform the initial Service Workbench installation from a deployment instance, before performing subsequent redeploys using the CI-CD pipeline.  Configuration changes are achieved by uploading edited configuration files to the relevant location in the ``artifacts`` bucket: e.g.: the main config file goes in the ``settings`` folder.

Read the files ``docs/docs/best_practices/cicd.md`` and ``main/cicd/README.md`` for an explanation of the two stages ``cicd-pipeline`` and ``cicd-source``.   Only ``cicd-pipeline`` is necessary to deploy Service Workbench directly from your GitHub repo; ``cicd-code`` creates another repository in CodeCommit.

The sequence to follow is:

* Create a config file in ``main/cicd/cicd-pipeline/config/settings/`` by copying ``example-github.yml`` and naming it ``<stage>.yml``.
* Edit the config file with the details of your GitHub repo.
    * The **awsProfile** value refers to the profile on the build instance used by CodeBuild; use the value ``awsProfile: default``.  

.. note::
    If you define the ``awsProfile`` value in the ``cicd-pipeline`` config file, you must **not** define the ``awsProfile`` value in the Service Workbench main configuration file (``main/config/settings/<stage.yml``).  Comment out this line.

Once configured, commits to the GitHub repo specified above will trigger a CodePipeline pipeline which will retrieve the code from GitHub, and deploy Service Workbench using a CodeBuild build project.  You will need to provide access to your GitHub repo using the **Connect to GitHub** mechanism within CodePipeline.
