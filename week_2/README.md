# WORKFLOW ORCHESTRATION

This is the process of tying together the many steps of a data pipeline (Extracting, transforming, loading, etc.), the tools that perform them and automating their execution.

### KESTRA

Kestra is a workflow orchestration tool similar to Apache Airflow.

As opposed to *automation* which is the execution of individual task without manual intervention, *orchestration* automates the flow of multiple interconnected tasks and processes, ensuring each is executed in the correct order with the right dependenncies etc. A common use case is an ETL data pipeline.

### FLOWS

A flow is a yaml file where you define the orchestration of a set of tasks. Lets take *01_getting_started_data_pipeline* and break down a few key components.

**Inputs**: You can define inputs that will modify the execution of your flow in some way. Arguments include
- id: name of the variable
- type: how the user modifies the variable (SELECT to pick one, ARRAY to make a list, etc)
- itemType: define the data type of the variable
- displayName: how it is described to the user executing the flow
- values: the available options for SELECT
- defaults: default value

Can be accessed with "inputs.**input id**"

**Tasks**: The actual tasks you are orchestrating. Can be executing scripts, performing SQL queries, etc.

When you execute a flow, you can observe the timing and results of each task and subtask in Kestra's Gantt tab. In the output tab, you can see the output each task may have had, such as the results of an SQL query.

**Variables**: Variables that can be defined dynamically during execution, as values from either input or tasks. Defined as "*variable name*: *"value*