# POC_Snowflake_DBT_INGESTION
Welcome to the Proof of Concept (POC) for Snowflake’s dbt ingestion feature! This is your chance to try it early, see how it works with your dbt project, and help us improve before general release.

With this POC, you’ll take your dbt manifest.json file, run a stored procedure in Snowflake, and instantly get ready-to-use semantic model YAML file that you can copy into your account. If you’re not using MetricFlow, we’ll just output an object of tables, columns, and descriptions.

Note: This repository is a shortened version of the helper-snowparser-for-dbt, which is part of the emerging-solutions-toolbox in the Snowflake Labs, and can be found [here](https://github.com/Snowflake-Labs/emerging-solutions-toolbox/tree/main/helper-snowparser-for-dbt).

**What You’ll Get**
* A Snowflake Semantic model generated directly from your dbt project
* Automatic extraction of tables, columns, and relationships
* Rich metrics and relationships if using MetricFlow
* A repeatable process you can run again whenever your dbt project changes

**Prerequisites**
Before you start, please make sure you have:
1. Downloaded the dbt `manifest.json` file from your dbt project
2. Created a Snowflake trial account with ADMINUSER rights, or else permissions to:
     * Create and write to a stage in Snowflake
     * Create a stored procedure
  
**Step-by-Step Instructions** 
1. Deploy Stored Procedure
* Copy and paste contents of setup.sql in a Snowflake SQL Worksheet
* Run the entire script

2. Upload Your Manifest
* In the Snowsight navigation bar, go to Database Explorer > GENAI_UTILITIES > UTILITIES > Stages > DROPBOX and upload your `manifest.json` by clicking on the blue "+Files" button in the upper right corner

3. Run the Stored Procedure
In a Snowsight SQL Worksheet, execute the Stored Procedure passing the staged manifest path as a parameter

* If you are using MetricFlow semantic models (remove the last lines if you don't want to use the manifest data types):
```
CALL SNOWPARSER_DBT_SEMANTIC_YAML(
    manifest_file => '@DROPBOX/manifest.json',
    semantic_view_name => 'test_semantic_view_fixed',
    semantic_view_description => 'Test with parse_snowflake_columns=FALSE',
    parse_snowflake_columns => FALSE  
);
```

* If you are NOT using MetricFLow semantic models, run the following code (note: you can leave the array empty, if you want to process all models found in your file):
```
CALL SNOWPARSER_DBT_GET_OBJECTS(
    manifest_file => '@DROPBOX/manifest.json', 
    dbt_models =>  TO_ARRAY(['customers','order_items', 'orders', 'stg_locations', 'stg_products']) 
);

```

4. Save and Upload the YAML file
* The stored procedure will parse your dbt file and return a semantic YAML file (if your dbt project has semantic models) or a list of tables and columns otherwise.
* Copy that output and create a YAML file with this content on your computer. Check that your tables, columns, metrics and other elements look correct or adjust if necessary.
* Next, upload this file to the DROPBOX stage as well (check the steps under 2 if needed).


5. Create a Semantic View
* In the Snowsight navigation bar, go to AI & ML > Cortex Analyst 
* Select the GENAI_UTILITIES database in the top and the utilities schema, then select the DROPBOX stage
* You will find your uploaded YAML file under the tab 'Semantic models'
* Click on the three horizontal dots on the right of the row and select 'Convert to Semantic View'
* Select the location to store the Semantic View (e.g., database GENAI_UTILITIES, schema UTILITIES)
* Note: this last step only works, if the underlying tables for the semantic view have been uploaded to the same database
* Alternative approach: Snowflake Semantic YAMLs can also be converted to native Semantic Views with the [CREATE_SEMANTIC_VIEW_FROM_YAML](https://docs.snowflake.com/en/sql-reference/stored-procedures/system_create_semantic_view_from_yaml) function

