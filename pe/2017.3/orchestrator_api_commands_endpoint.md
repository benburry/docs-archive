# Puppet orchestrator API: commands endpoint

Use the `/command/deploy`, `/command/task`, and `/command/stop` endpoints to start and stop orchestrator jobs.

## POST /command/deploy

Run the orchestrator across all nodes in an environment.

### Request format

The request body must be a JSON object using these keys:

|Key|Definition|
|---|----------|
|`environment`|The environment to deploy. **This key is required.**|
|`description`|A description of the job.|
|`noop`|Whether to run the agent in no-op mode. The default is `false`.|
|`no_noop`|Whether to run the agent in enforcement mode. Defaults to `false`. This flag overrides `noop = true` if set in the agent's `puppet.conf`, and cannot be set to true at the same time as the `noop` flag.|
|`concurrency`|The maximum number of nodes to run at once. The default, if unspecified, is unlimited.|
|`scope`|The PuppetDB query, a list of nodes, or an application/application instance to deploy.|
|`enforce_environment`|Whether to force agents to run in the same environment in which their assigned applications are defined.|
|`debug`|Whether to use the `--debug` flag on Puppet agent runs.|
|`trace`|Whether to use the `--trace` flag on Puppet agent runs.|
|`evaltrace`|Whether to use the `--evaltrace` flag on Puppet agent runs.|
|`target`|The name of an application or application instance to deploy. If an application is specified, all instances of that application will be deployed. If this key is left blank or unspecified without a scope, the entire environment will be deployed. **This key is deprecated.**|

For example, to deploy the `production` environment in no-op mode, the following request is valid:

```json
{
  "environment" : "production",
  "description" : "test run",
  "noop" : true,
  "scope" : {
    "whole_environment" : true
  }
}
```

### Scope

Scope is a JSON object containing exactly one of these keys:

|Key|Definition|
|---|----------|
|`application`|The name of an application or application instance to deploy. If an application type is specified, all instances of that application will be deployed.|
|`nodes`|A list of node names to target.|
|`query`|A PuppetDB or PQL query to use to discover nodes. The target is built from `certname` values collected at the top level of the query.|
|`whole_environment`|A boolean to deploy all nodes in an environment \(assigned via the node classifier\) or included in applications in that environment. **This key is deprecated and will be removed in a future version of the API.**|

To deploy an application instance in the `production` environment, the following request is valid:

```json
{
  "environment" : "production",
  "scope" : {
    "application" : "Wordpress_app[demo]"
  }
}
```

To deploy a list of nodes, the following request is valid:

```json
{
  "environment" : "production",
  "scope" : {
    "nodes" : ["node1.example.com", "node2.example.com"]
  }
}
```

To deploy a list of nodes with the `certname` value matching a regex, the following request is valid:

```json
{
  "environment" : "production",
  "scope" : {
    "query" : ["from", "nodes", ["~", "certname", ".*"]]
  }
}
```

### Response format

If all node runs succeed, and the environment is successfully deployed, the server returns a `202` response.

The response will be a JSON object containing a link to retrieve information about the status of the job and uses any one of these keys:

|Key|Definition|
|---|----------|
|`id`|An absolute URL that links to the newly created job.|
|`name`|The name of the newly created job.|

For example:

```json
{
  "job" : {
    "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234"
    "name" : "1234"
  }
}
```

### Error responses

See the error response documentation for the general format of error responses. For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/unknown-environment`|If the environment does not exist, the server returns a `404` response.|
|`puppetlabs.orchestrator/empty-environment`|If the environment requested contains no applications or no nodes, the server returns a `400` response.|
|`puppetlabs.orchestrator/empty-target`|If the application instance specified to deploy does not exist or is empty, the server returns a `400` response.|
|`puppetlabs.orchestrator/dependency-cycle`|If the application code contains a cycle, the server returns a `400` response.|
|`puppetlabs.orchestrator/puppetdb-error`|If the orchestrator is unable to make a query to PuppetDB, the server returns a `400` response.|
|`puppetlabs.orchestrator/query-error`|If a user does not have appropriate permissions to run a query, or if the query is invalid, the server returns a `400` response.|

## POST /command/task

Run a task job across a set of nodes.

### Request format

The request body must be a JSON object and uses the following keys:

|Key|Definition|
|---|----------|
|`environment`|The environment to load the task from. The default is `production`.|
|`scope`|The PuppetDB query or a list of targets. Application scopes are not allowed for task jobs. **This key is required.**|
|`description`|A description of the job.|
|`noop`|Whether to run the job in no-op mode. The default is `false`.|
|`task`|The task to run on the targets. **This key is required.**|
|`params`|The params to pass to the task. This key is required, but can be an empty object.|

For example, to run the `package` task on `node1.example.com`, the following request is valid: 

```
{
  "environment" : "test-env-1",
  "task" : "package",
  "params" : {
    "action" : "install",
    "name" : "httpd"
  },
  "scope" : {
    "nodes" : ["node1.example.com"]
  }
}
```

### Scope

Scope is a JSON object containing exactly one of the following keys:

|Key|Definition|
|---|----------|
|`nodes`|An array of node names to target.|
|`query`|A PQL query to use to discover nodes. The target is built from the certname values collected at the top level of the query.|

To run a task on a list of nodes, the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "nodes" : ["node1.example.com", "node2.example.com"]
  }
}
```

To run a task on a list of nodes with the certname value matching a regex, the following request is valid:

```
{
  "environment" : "production",
  "scope" : {
    "query" : ["from", "nodes", ["~", "certname", ".*"]]
  }
}
```

### Response format

If the task is starts successfully, the response will have a `202` status.

The response will be a JSON object containing a link to retrieve information about the status of the job. The keys of this object are:

-   `id`: an absolute URL that links to the newly created job.
-   `name`: the name of the newly created job. For example:

    ```
    {
      "job" : {
        "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234",
        "name" : "1234"
      }
    }
    ```


## POST /command/stop

Stop an orchestrator job that is currently in progress. Puppet agent runs that are in progress will finish, but no new agent runs will start. While agents are finishing, the server will continue to produce events for the job.

The job will transition to status `stopped` once all pending agent runs have finished.

This command is *idempotent*: it can be issued against the same job any number of times.

### Request format

The JSON body of the request must contain the ID of the job to stop. The job ID is the same value as the `name` property returned with the `deploy` command.

-   `job`: the name of the job to stop.

For example:

```json
{
  "job": "1234"
}
```

### Response format

If the job is stopped successfully, the server returns a `202` response. The response uses these keys:

|Key|Definition|
|---|----------|
|`id`|An absolute URL that links to the newly created job.|
|`name`|The name of the newly created job.|
|`nodes`|A hash that shows all of the possible node statuses, and how many nodes are currently in that status.|

For example:

```json
{
  "job" : {
    "id" : "https://orchestrator.example.com:8143/orchestrator/v1/jobs/1234",
    "name" : "1234",
    "nodes" : {
      "new" : 5,
      "running" : 8,
      "failed"  : 3,
      "errored" : 1,
      "skipped" : 2,
      "finished": 5
    }
  }
}
```

### Error responses

See the error response documentation for the general format of error responses. For this endpoint, the `kind` key of the error displays the conflict.

|Key|Definition|
|---|----------|
|`puppetlabs.orchestrator/validation-error`|If a job name is not valid, the server returns a `400` response.|
|`puppetlabs.orchestrator/unknown-job`|If a job name is unknown, the server returns a `404` response.|

