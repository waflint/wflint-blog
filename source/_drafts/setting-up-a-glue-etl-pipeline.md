---
title: setting-up-a-glue-etl-pipeline
categories: "Data Engineering"
tags: ["AWS Glue", "Python", "Relational Databases", ]
---

Recently I did some work setting up pipelines for a new project and was given some interesting constraints on top of the relatively straightforward requirements

The pipeline(s) in question were designed to route data from tables stored on a Redshift cluster across to a SQLServer RDS, but in a way that would be maintainable by analysts rather than engineers!

Not only were we aiming for robust operation, but something where the orchestration and logical units were actionable without deep subject matter expertise.


## Networking

### S3 Connectivity

### Working Between Databases

on top of the initial security group [setup](https://docs.aws.amazon.com/glue/latest/dg/setup-vpc-for-glue-access.html)

a glue ETL job is capable of supporting multiple connections for JDBC sources, however the network interface used for the job is instantiated based on only the _first_ connection.

to simplify, if you're attempting to connect to multiple databases in the same glue job, they must be in both the same VPC _and_ the same subnet.

## SQLServer quirks

### Shell Scripts and External Libraries

## Final Flowchart

{% mermaid flowchart LR %}
subgraph vpcone [Dev Account]
    I[CRON TIMER] --> B
    subgraph one [Subnet 1]
        A[("Source  
        Database")] --> B{"Glue  
        Unload"}
    end
    B --> S[/"Dev Bucket"/]
    S --> C
    subgraph two [Subnet 2]
        direction TB
        C{"Glue  
        Transform"} -->T[("Dev App  
        RDS")] -->Q["Dev  
        Server"]
    end
end
subgraph vpctwo [Prod Account]
    J[Bucket Trigger] -->U
    C --> V[/Prod Bucket/]
    V --> U
    subgraph three [subnet 3]
        U{"Glue  
        Transform"} --> X[("Prod App  
        RDS")] 
        X --> Y["Prod  
        Server"]
    end
end
{% endmermaid %}