# Materialized View Stage

The Coalesce Materialized View Stage UDN is a versatile node that allows you to develop and deploy a Materialized View in Snowflake.

A [materialized view](https://docs.snowflake.com/en/user-guide/views-materialized) is a pre-computed data set derived from a query specification (the SELECT in the view definition) and stored for later use. materialized views can speed up expensive aggregation, projection, and selection operations, especially those that run frequently and that run on large data sets.

## Version Information
 * UDN Name: Materialized View Stage
 * Package Name: 

 * Certified Version: 1.0
 * Documentation Version: 1.0
 * Current Version: 1.0
 * Originally Developed by: John Gontarz
 * Current Owner: John Gontarz
 * Materialization Type: Materialized View
 * Create Template: Yes
 * Run Template: No

 * Deployment Strategy: Advanced
 * Deployment Method: ALTER when possible otherwise CREATE OR REPLACE

 * Test Enabled: No

## Node Configuration
The Materialized View Stage has three configuration groups:

* [Node Properties](#node-properties)
* [Materialized View Options](#materialized-view-options)
* [General Options](#general-options)

### Node Properties
There are four configs within the **Node Properties** group.

* **Storage Location**: Storage Location where the Materialized View will be created.
* **Node Type**: Name of template used to create node objects.
* **Description**: A description of the node's purpose.
* **Deploy Enabled**:
  * If TRUE the node will be deployed / redeployed when changes are detected.
  * If FALSE the node will not be deployed or will be dropped during redeployment.

### Materialized View Options

There are two configs within the **Materialized View Options** group.

* **Cluster key**: True/False to determine whether Materialized view is to be clustered or not
    * True – Allows you to be specify the column based on which clustering is to be done.
           -Allow Expressions Cluster Key-True ->allows to add an expression to the specified cluster key
    * False – No clustering done
* **Secure**: True / False Toggle to determine whether Materialized view to be created in a secured mode
    * True – Materialized view created in a secured mode
    * False – No additional secure option added during Materialized view creation

### General Options

* **Distinct**: True / False toggle that specifies whether or not to return DISTINCT rows.
  
## Limitations of Materialized view

Materialized view has a set of limitations as listed here

* A materialized view cannot query a materialized or non-materialized view
* A materialized view does not support group by all but group by is supported
* A materialized view can query only a single table.

This link provides insights on more [limitations](https://docs.snowflake.com/en/user-guide/views-materialized#limitations-on-creating-materialized-views).

## Deployment

### Initial Deployment
When deployed for the first time into an environment Materialized View will execute three stages:

* **Create Materialized View**
This stage will execute a CREATE OR REPLACE statement and create a Materialized View in the target environment.

* **Applying Materialized View Clustering**
This stage will apply clustering to the created Materialized View if Clustering is set to true

* **Resume recluster Materialized View**
This stage will resume the Materialized View based on clustering

### Redeployment

After the Materialized View has deployed for the first time into a target environment, subsequent deployments may result in either altering the Materialized View or recreating the Materialized View.

If a Materialized View is to be altered this will run the following stage:

* **Alter Materialized View**
  * This stage will execute an ALTER statement and alter the Materialized View in the target environment setting the new parameters.
  
#### Altering the Materialized View
  
There are two config changes that if made in isolation or all-together will result in an ALTER statement to modify the Materialized View in the target environment.

* Cluster key
* Secure

#### Recreating the Materialized View

If anything changes other than the configuration options specified above then the Materialized View will be recreated by running a CREATE OR REPLACE statement.


### Undeployment

If a Materialized View is deleted from a Workspace, that Workspace is committed to Git and that commit deployed to a higher-level environment then the Materialized View in the target environment will be dropped.

This is executed as a single stage:

* Drop Materialized View

# Code

*  [Node definition](https://github.com/coalesceio/Materialized-View-Node/blob/main/nodeTypes/MaterializedViewStage-178/definition.yml)
*  [Create Template](https://github.com/coalesceio/Materialized-View-Node/blob/main/nodeTypes/MaterializedViewStage-178/create.sql.j2)

[<img src="https://github.com/coalesceio/Materialized-View-Node/assets/7216836/adc00d67-6a90-4bf9-b944-ba723cce461a" alt="Git Logo" height="40">](https://github.com/coalesceio/Materialized-View-Node)
