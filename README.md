# Materialized View Stage

The Coalesce Materialized View Stage UDN is a versatile node that allows you to develop and deploy a Materialized View in Snowflake.

A [materialized view](https://docs.snowflake.com/en/user-guide/views-materialized) is a pre-computed data set derived from a query specification (the SELECT in the view definition) and stored for later use. materialized views can speed up expensive aggregation, projection, and selection operations, especially those that run frequently and that run on large data sets.

## Node Configuration

The Materialized View Stage has three configuration groups:

* [Node Properties](#node-properties)
* [Materialized View Options](#materialized-view-options)
* [General Options](#general-options)

### Node Properties
There are four configs within the **Node Properties** group.

| **Property** | **Description** |
|-------------|-----------------|
| **Storage Location** | Storage Location where the Materialized View will be created |
| **Node Type** | Name of template used to create node objects |
| **Description** | A description of the node's purpose |
| **Deploy Enabled** | If TRUE the node will be deployed / redeployed when changes are detected<br/>If FALSE the node will not be deployed or will be dropped during redeployment |

### Materialized View Options

There are two configs within the **Materialized View Options** group.

| **Option** | **Description** |
|------------|----------------|
| **Cluster key** | True/False to determine whether Materialized view is to be clustered or not<br/>- **True**: Allows you to specify the column based on which clustering is to be done<br/>  -**Allow Expressions Cluster Key**: True allows to add an expression to the specified cluster key<br/>- **False**: No clustering done |
| **Secure** | True / False Toggle to determine whether Materialized view to be created in a secured mode<br/>- **True**: Materialized view created in a secured mode<br/>- **False**: No additional secure option added during Materialized view creation |

### General Options

| **Option** | **Description** |
|------------|----------------|
| **Distinct** | True / False toggle that specifies whether or not to return DISTINCT rows |

## Limitations of Materialized view

Materialized view has a set of limitations:

* A materialized view cannot query a materialized or non-materialized view
* A materialized view does not support group by all but group by is supported
* A materialized view can query only a single table.

Review [Snowflake's Limitations on Creating Materialized Views](https://docs.snowflake.com/en/user-guide/views-materialized#limitations-on-creating-materialized-views).

## Deployment

### Initial Deployment
When deployed for the first time into an environment Materialized View will execute three stages:

| **Stage** | **Description** |
|-----------|----------------|
| **Create Materialized View** | This stage will execute a CREATE OR REPLACE statement and create a Materialized View in the target environment |
| **Applying Materialized View Clustering** | This stage will apply clustering to the created Materialized View if Clustering is set to true |
| **Resume recluster Materialized View** | This stage will resume the Materialized View based on clustering |

### Redeployment

After the Materialized View has deployed for the first time into a target environment, subsequent deployments may result in either altering the Materialized View or recreating the Materialized View.

If a Materialized View is to be altered this will run the following stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Alter Materialized View** | This stage will execute an ALTER statement and alter the Materialized View in the target environment setting the new parameters |
  
#### Altering the Materialized View
  
There are two config changes that if made in isolation or all-together will result in an ALTER statement to modify the Materialized View in the target environment.

* Cluster key
* Secure

#### Recreating the Materialized View

If anything changes other than the configuration options specified above then the Materialized View will be recreated by running a CREATE OR REPLACE statement.

### Redeployment with no changes 

If the nodes are redeployed with no changes compared to previous deployment,then no stages are executed

#### Node Type Switching

Node Type switching is supported starting from Coalesce version **7.29+**.

From this version onward, a node’s materialization type can be switched from one supported type to another, subject to certain limitations.

For more info click here - [Node Type Switching Logic and Limitations](#node-type-switching-logic)

### Undeployment

If a Materialized View is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the Materialized View in the target environment will be dropped.

This is executed as a single stage:

| **Stage** | **Description** |
|-----------|----------------|
| **Drop Materialized View** | Removes the materialized view |

-----------------

#### Node Type Switching Logic
| Current MaterializationType | Desired MaterializationType | Stage |
|------------|--------|-------|
| Table | Table | 1. Warning (if applicable)<br/>2. Metadata Update(if applicable)<br/>3. Alter |
| Transient Table | TransientTable | 1. Warning (if applicable)<br/>2. Metadata Update(if applicable)<br/>3. Alter |
| View | View | 1. Warning (if applicable)<br/>2. Create |
| Any Other | Table | 1. Warning (if applicable)<br/>2. Drop <br/> 3. Create |
| Any Other | View | 1. Warning (if applicable)<br/>2. Drop <br/> 3. Create |
| Any Other | Transient Table | 1. Warning (if applicable)<br/>2. Drop <br/> 3. Create |

Please review the documented limitations before performing a node type switch to ensure compatibility and avoid unintended deployment issues.

#### ⚠ Limitations of Node Type Switching (Current)

| # | Current Materialization | Desired Materialization | Limitation |
|---|--------------------------|--------------------------|------------|
| 1 | Older Version Iceberg Table | Table | Results in `ALTER` failure. Iceberg tables require `ALTER ICEBERG TABLE`. Works only if latest package (with switching support) is already used. |
| 2 | Older Version Create or Alter View | Any | Switch fails unless current node uses latest package supporting node type switching. |
| 3 | First Node in Pipeline | Any | Not supported. First node is foundational and switching may disrupt the pipeline. |
| 4 | External Packages | Any | Not supported as they typically act as first nodes in the pipeline. |
| 5 | Functional Packages | Any | Not supported due to column re-sync behavior which may cause schema inconsistencies. |
| 6 | Dynamic Dimension / LRV | Any | System columns must be manually dropped before redeployment. |
| 7 | Any | Any Other | After performing node switching, the `Create/Run` in Workspace browser may not work as expected due to changes in the node’s materialization type. |

--------------

## Code

* [Node definition](https://github.com/coalesceio/Materialized-View-Node/blob/main/nodeTypes/MaterializedViewStage-178/definition.yml)
* [Create Template](https://github.com/coalesceio/Materialized-View-Node/blob/main/nodeTypes/MaterializedViewStage-178/create.sql.j2)
