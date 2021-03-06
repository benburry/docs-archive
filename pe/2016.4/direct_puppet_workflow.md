---
layout: default
title: "Direct Puppet: a workflow for controlling change"
canonical: "/pe/latest/direct_puppet_workflow.html"
---

The Puppet orchestrator---used alongside other PE tools, such as Code Manager---allows you to control when and how infrastructure changes are made before they reach your production environment.

The Direct Puppet workflow gives you precise control over rolling out changes, from updating data and classifying nodes, to deploying new Puppet code. In this workflow, you configure your Puppet agents to use cached catalogs during scheduled runs, and you send new catalogs only when you're ready, via Puppet orchestrator jobs. Scheduled runs will continue to enforce the desired state of the last orchestration job until you send another new catalog.

## Direct Puppet workflow

In the Direct Puppet workflow, you set a up a node group for testing and validating code on a feature branch before you merge and promote it into your production environment.

**Assumptions and prerequisites:**

- To use this workflow, you must [prepare your Puppet agents for orchestration services](./orchestrator_install.html#prepare-your-agents-for-orchestration-services) so that they enforce cached catalogs by default and compile new catalogs only when instructed to by Puppet orchestration jobs.

- This workflow also assumes you're familiar with [Code Manager](./cmgmt_managing_code.html). It involves making changes to your Puppet control repo---adding or updating modules, editing manifests, or changing your Hiera data. You'll also run deploy actions from the Code Manager command line tool and the Puppet orchestrator, so ensure you have access to a host with [PE client tools installed](./install_pe_client_tools.html).


### Set up a node group for testing new features

1. [Create an environment node group for branch testing](./console_classes_groups.html#creating-environment-node-groups).

2. Set the **Parent name** to the environment group that contains the nodes you want to test. In this workflow, we'll use **Production environment**.

3. Set the **Environment** to **agent-specified** and select **Environment group**.

   <a href="./images/direct_puppet_create_environment.png"><img src="./images/direct_puppet_create_environment.png" alt="Create Environment"></a>

### Create a rule to match the nodes that you want to test

1. In the **Rules** tab of the branch testing environment group, add the following rule:

   Option | Value
   -------|------
   Fact   | `agent_specified_environment`
   Operator | `~`
   Value | `^.+`

This rule matches any nodes from the parent group that have the `agent_specified_environment` fact set. By matching nodes to this group, you give the nodes permission to override the server-specified environment, and use their agent-specified environment instead.

### Create a feature branch

1. Branch your [control repository](./cmgmt_control_repo.html), and name the new branch. For example, 'my_feature_branch'.

   <a href="./images/direct_puppet_create_branch.png"><img src="./images/direct_puppet_create_branch.png" alt="Create Environment"></a>

### Make your changes

Make changes to the code on your feature branch. Commit and push the changes to the `my_feature_branch`.

### Deploy code to the Puppet master

1. To deploy the feature branch to the Puppet master, run the following Code Manager command:

   ~~~
   puppet code deploy --wait my_feature_branch
   ~~~

   <a href="./images/direct_puppet_deploy_feature.png"><img src="./images/direct_puppet_deploy_feature.png" alt="Create Environment"></a>

> **Note:** After this code deployment, there is a short delay while Puppet Server loads the new code.

### Test your changes

1. To run Puppet on a few agent-specified development nodes in the 'my_feature_branch' environment, run the following orchestrator command:

   ~~~
   puppet job run --nodes my-dev-node1,my-dev-node2 --environment my_feature_branch
   ~~~

   <a href="./images/direct_puppet_test_changes.png"><img src="./images/direct_puppet_test_changes.png" alt="Create Environment"></a>

### Validate your testing changes

Open the links in the orchestrator command output to review the node run reports in the console. Ensure the changes have the effect you intend.

### Merge and promote your code

1. If everything works as expected on the development nodes, and you're ready to promote your changes into production, merge `my_feature_branch` into the `production` branch in your control repo.

   <a href="./images/direct_puppet_promote.png"><img src="./images/direct_puppet_promote.png" alt="Create Environment"></a>

2. To deploy your updated `production` branch to the Puppet master, run the following Code Manager command:

   ~~~
   puppet code deploy --wait production
   ~~~

### Preview your changes

Before running Puppet across the whole production environment, preview the job by running the [`puppet job plan`](./orchestrator_job_plan.html) command. 

1. To ensure the job will capture all the nodes in the `production` environment, as well as the agent-specified development nodes that just ran with the `my_feature_branch` environment, use the following query as the job target:

   ~~~
   puppet job plan --query 'inventory {environment in ["production", "my_feature_branch"]}
   ~~~

### Run Puppet on the production environment

1. If you're satisfied with the changes and are ready to enforce the updated `production` environment across all the `production` nodes, run the orchestrator job.

   ~~~
   puppet job run --query 'inventory {environment in ["production", "my_feature_branch"]}
   ~~~

   <a href="./images/direct_puppet_run_production.png"><img src="./images/direct_puppet_run_production.png" alt="Create Environment"></a>

### Validate your production changes

Check the node run reports to confirm that the changes were applied as intended. If so, you're done!

Repeat this process as you develop and promote your code.






