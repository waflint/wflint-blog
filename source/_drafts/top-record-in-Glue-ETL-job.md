---
title: top record in Glue ETL job
date: 2023-06-13 13:16:55
categories: "Data Engineering"
tags: ["AWS Glue", "pyspark", "Data Engineering"]
description: A quick function to filter for the top record in a set in pyspark when working in AWS Glue ETL
---

Just a quick writeup, but one of my recent projects required filtering a dataframe down with a rank function and it took me a few minutes to figure out how best to accomplish this while ingesting data from a glue table.

<!-- more -->



{% codeblock lang:python %}
def MyTransform (glueContext, dfc) -> DynamicFrameCollection:
    from pyspark.sql import Window
    import pyspark.sql.functions as f

    tempdf = dfc.select(list(dfc.keys())[0]).toDF()

    w = Window.partitionBy("country_code").orderBy(f.desc("snapshot_datetime"))
    df = tempdf.withColumn("rn", f.row_number().over(w)).filter("rn = 1").drop("rn")

    dyf_filtered = DynamicFrame.fromDF(df, glueContext, "filtered_df")
    return(DynamicFrameCollection({"CustomTransform0": dyf_filtered}, glueContext))
{% endcodeblock %}

