

In #SAP #Datasphere, you have the need to write to a table created in the *OPENSQL* schema equivalent of the space
By default, datasphere operations like Data flow have *Read* only access to the table defined in the *OPENSQL* schema

> see https://help.sap.com/docs/SAP_DATASPHERE/be5967d099974c69b77f4549425ca4c0/7eaa370fe4624dea9f182ee9c9ab645f.html?locale=en-US


In order to GRANT *Write* access, a procedure *GRANT_PRIVILEGE_TO_SPACE* must be run by the *OPENSQL*  user

> running it with the Console user does not return an error, but it does nothing

For example suppose the creation of Space **IOT**, where we want to write to table **IOT#OPENSQL.ToolsData** with a data flow

We would grant *Insert* permission to space *IOT* with this procedure 

```sql
CALL "DWC_GLOBAL"."GRANT_PRIVILEGE_TO_SPACE" (

OPERATION => 'GRANT',

PRIVILEGE => 'INSERT',

SCHEMA_NAME => 'IOT#OPENSQL',

OBJECT_NAME => 'ToolsData',

SPACE_ID => 'IOT');
```

## Upsert

In case the Data flow is configured as *Upsert*, grant also the **DELETE** and **UPDATE** permission.