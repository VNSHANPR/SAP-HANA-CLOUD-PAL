![Alt text](images/logo_hc_ta.png?raw=true "Title")
## SAP HANA Predictive Appliance Library

In this excercise we will use S4 HANA Sales header Virtual tables in HANA Cloud and use it to load sales order history of a specific Material ,and 
then use this data along with Weather Data from Azure Data lake to Predict the future Sales of the Material, and see co-relation if any with the Weather Data.

### Query the S4HANA Sales header Virtual Tables VBAP,VBAK,MAKT in HANA CLOUD


```markdown

select  a.AUDAT as ID,sum(b.KWMENG) as "SALES_ORDER_QUANTITY" , DAYNAME(a.AUDAT) as DAY,CASE WHEN DAYNAME(a.AUDAT)='SUNDAY' THEN 1 WHEN DAYNAME(a.AUDAT)='SATURDAY' THEN 1 ELSE 0 END AS WEEKEND,MONTH(a.AUDAT) as MONTH,b.MATNR,c.MAKTG from 
"DBADMIN"."VT_VBAK" a inner join "DBADMIN"."VT_VBAP" b ON a.VBELN=b.VBELN and a.MANDT=b.MANDT inner join  "DBADMIN"."VT_MAKT" c ON c.MATNR=b.MATNR 
where b.MATNR='MR314201' and c.SPRAS='E' group by b.MATNR,a.AUDAT,c.MAKTG order by a.AUDAT DESC
```


### Load filtered S4HANA data (only about 600 rows) into a table
Ignore below command if the table already exists!
```markdown
CREATE TABLE S4_MAT_TRAIN as (select  a.AUDAT as ID,sum(b.KWMENG) as "SALES_ORDER_QUANTITY" , DAYNAME(a.AUDAT) as DAY,CASE WHEN DAYNAME(a.AUDAT)='SUNDAY' THEN 1 WHEN DAYNAME(a.AUDAT)='SATURDAY' THEN 1 ELSE 0 END AS WEEKEND,MONTH(a.AUDAT) as MONTH,b.MATNR,c.MAKTG from 
"DBADMIN"."VT_VBAK" a inner join "DBADMIN"."VT_VBAP" b ON a.VBELN=b.VBELN and a.MANDT=b.MANDT inner join  "DBADMIN"."VT_MAKT" c ON c.MATNR=b.MATNR 
where b.MATNR='MR314201' and c.SPRAS='E' group by b.MATNR,a.AUDAT,c.MAKTG order by a.AUDAT DESC);
```

### Create PAL Regression Parameter Table


```markdown

CREATE LOCAL TEMPORARY COLUMN TABLE 
	#PAL_PARAMETER_TBL 
	("PARAM_NAME" VARCHAR(256), "INT_VALUE" INTEGER, "DOUBLE_VALUE" DOUBLE, "STRING_VALUE" VARCHAR(1000));
INSERT INTO #PAL_PARAMETER_TBL VALUES ('THREAD_RATIO',NULL,0.5,NULL);
INSERT INTO #PAL_PARAMETER_TBL VALUES ('PMML_EXPORT',2,NULL,NULL);
INSERT INTO #PAL_PARAMETER_TBL VALUES ('CATEGORICAL_VARIABLE',NULL,NULL,'DAY');
INSERT INTO #PAL_PARAMETER_TBL VALUES ('CATEGORICAL_VARIABLE',NULL,NULL,'WEEKEND');
INSERT INTO #PAL_PARAMETER_TBL VALUES ('CATEGORICAL_VARIABLE',NULL,NULL,'MONTH');
```

### Run PAL Regression Algorithim 

```markdown

CALL _SYS_AFL.PAL_LINEAR_REGRESSION(S4_MAT_TRAIN,"#PAL_PARAMETER_TBL", ?, ?, ?, ?,?);

```

