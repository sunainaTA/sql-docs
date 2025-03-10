---
title: Elastic Database Jobs (preview)
description: "Configure Elastic Database Jobs (preview) to run Transact-SQL (T-SQL) scripts across a set of one or more databases in Azure SQL Database"
author: srinia
ms.author: srinia
ms.reviewer: wiassaf, mathoma
ms.date: 05/03/2022
ms.service: sql-database
ms.subservice: elastic-jobs
ms.topic: conceptual
ms.custom:
  - "seo-lt-2019"
  - "sqldbrb=1"
---
# Create, configure, and manage elastic jobs (preview)
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

In this article, you will learn how to create, configure, and manage elastic jobs.

If you have not used Elastic jobs, [learn more about the job automation concepts in Azure SQL Database](job-automation-overview.md).

## Create and configure the agent

1. Create or identify an empty S1 or higher database. This database will be used as the *Job database* during Elastic Job agent creation.
2. Create an Elastic Job agent in the [portal](https://portal.azure.com/#create/Microsoft.SQLElasticJobAgent) or with [PowerShell](elastic-jobs-powershell-create.md#create-the-elastic-job-agent).

   ![Creating Elastic Job agent](./media/elastic-jobs-overview/create-elastic-job-agent.png)

## Create, run, and manage jobs

1. Create a credential for job execution in the *Job database* using [PowerShell](elastic-jobs-powershell-create.md) or [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-credential-for-job-execution).
2. Define the target group (the databases you want to run the job against) using [PowerShell](elastic-jobs-powershell-create.md) or [T-SQL](elastic-jobs-tsql-create-manage.md#create-a-target-group-servers).
3. Create a job agent credential in each database the job will run [(add the user (or role) to each database in the group)](logins-create-manage.md). For an example, see the [PowerShell tutorial](elastic-jobs-powershell-create.md).
4. Create a job using [PowerShell](elastic-jobs-powershell-create.md) or [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
5. Add job steps using [PowerShell](elastic-jobs-powershell-create.md) or [T-SQL](elastic-jobs-tsql-create-manage.md#deploy-new-schema-to-many-databases).
6. Run a job using [PowerShell](elastic-jobs-powershell-create.md#run-the-job) or [T-SQL](elastic-jobs-tsql-create-manage.md#begin-unplanned-execution-of-a-job).
7. Monitor job execution status using the portal, [PowerShell](elastic-jobs-powershell-create.md#monitor-status-of-job-executions) or [T-SQL](elastic-jobs-tsql-create-manage.md#monitor-job-execution-status).

   ![Portal](./media/elastic-jobs-overview/elastic-job-executions-overview.png)

## Credentials for running jobs

Jobs use [database scoped credentials](/sql/t-sql/statements/create-database-scoped-credential-transact-sql) to connect to the databases specified by the target group upon execution. If a target group contains servers or pools, these database scoped credentials are used to connect to the `master` database to enumerate the available databases.

Setting up the proper credentials to run a job can be a little confusing, so keep the following points in mind:

- The database scoped credentials must be created in the *Job database*.
- **All target databases must have a login with [sufficient permissions](/sql/relational-databases/security/permissions-database-engine) for the job to complete successfully** (`jobuser` in the diagram below).
- Credentials can be reused across jobs, and the credential passwords are encrypted and secured from users who have read-only access to job objects.

The following image is designed to assist in understanding and setting up the proper job credentials. **Remember to create the user in every database (all *target user dbs*) the job needs to run**.

![Elastic Jobs credentials](./media/elastic-jobs-overview/job-credentials.png)

## Security best practices

A few best practice considerations for working with Elastic Jobs:

- Limit usage of the APIs to trusted individuals.
- Credentials should have the least privileges necessary to perform the job step. For more information, see [Authorization and Permissions](/dotnet/framework/data/adonet/sql/authorization-and-permissions-in-sql-server).
- When using a server and/or pool target group member, it is highly suggested to create a separate credential with rights on the `master` database to view/list databases that is used to expand the database lists of the server(s) and/or pool(s) prior to the job execution.

## Agent performance, capacity, and limitations

Elastic Jobs use minimal compute resources while waiting for long-running jobs to complete.

Depending on the size of the target group of databases and the desired execution time for a job (number of concurrent workers), the agent requires different amounts of compute and performance of the *Job database* (the more targets and the higher number of jobs, the higher the amount of compute required).

### Prevent jobs from reducing target database performance

To ensure resources aren't overburdened when running jobs against databases in a SQL elastic pool, jobs can be configured to limit the number of databases a job can run against at the same time.

Set the number of concurrent databases a job runs on by setting the `sp_add_jobstep` stored procedure's `@max_parallelism` parameter in T-SQL.


### Known limitations

These are the current limitations to the Elastic Jobs service.  We're actively working to remove as many of these limitations as possible.

| Issue | Description |
| :---- | :--------- |
| The Elastic Job agent needs to be recreated and started in the new region after a failover/move to a new Azure region. | The Elastic Jobs service stores all its job agent and job metadata in the jobs database. Any failover or move of Azure resources to a new Azure region will also move the jobs database, job agent and jobs metadata to the new Azure region. However, the Elastic Job agent is a compute only resource and needs to be explicitly re-created and started in the new region before jobs will start executing again in the new region. Once started, the Elastic Job agent will resume executing jobs in the new region as per the previously defined job schedule. |
| Concurrent jobs limit. | Currently, the preview is limited to 100 concurrent jobs. |
| Excessive Audit logs from Jobs database | The Elastic Job agent operates by constantly polling the Job database to check for the arrival of new jobs and other CRUD operations. If auditing is enabled on the server that houses a Jobs database, a large amount of audit logs may be generated by the Jobs database. This can be mitigated by filtering out these audit logs using the `Set-AzSqlServerAudit` command with a predicate expression.<BR/><BR/>For example:<BR/> `Set-AzSqlServerAudit -ResourceGroupName "ResourceGroup01" -ServerName "Server01" -BlobStorageTargetState Enabled -StorageAccountResourceId "/subscriptions/7fe3301d-31d3-4668-af5e-211a890ba6e3/resourceGroups/resourcegroup01/providers/Microsoft.Storage/storageAccounts/mystorage" -PredicateExpression "database_principal_name <> '##MS_JobAccount##'"`<BR/><BR/>This command will only filter out Job Agent to Jobs database audit logs, not Job Agent to any target databases audit logs.|
| Private endpoints are not supported | Elastic Job Agents currently cannot connect to Databases and Elastic Pools which restrict connections to private endpoints.|
| Use of a Hyperscale database as *Job database* | Using a Hyperscale database as a *Job database* isn't supported. However, elastic jobs can target Hyperscale databases in the same way as any other database in Azure SQL Database.|
| Serverless DBs and auto-pausing with Elastic Jobs. | When the job database is a serverless database, databases targeted by elastic jobs support auto-pausing, but will be resumed by job connections. 

## Best practices for creating jobs

Consider the following best practices when working with Elastic Database jobs:

### Idempotent scripts
A job's T-SQL scripts must be [idempotent](https://en.wikipedia.org/wiki/Idempotence). **Idempotent** means that if the script succeeds, and it is run again, the same result occurs. A script may fail due to transient network issues. In that case, the job will automatically retry running the script a preset number of times before desisting. An idempotent script has the same result even if its been successfully run twice (or more).

A simple tactic is to test for the existence of an object before creating it. A hypothetical example is shown below:

```sql
IF NOT EXISTS (SELECT * FROM sys.objects WHERE [name] = N'some_object')
    print 'Object does not exist'
    -- Create the object
ELSE
    print 'Object exists'
    -- If it exists, drop the object before recreating it.
```

Similarly, a script must be able to execute successfully by logically testing for and countering any conditions it finds.

## Next steps

- [Create and manage Elastic Jobs using PowerShell](elastic-jobs-powershell-create.md)
- [Create and manage Elastic Jobs using Transact-SQL (T-SQL)](elastic-jobs-tsql-create-manage.md)
