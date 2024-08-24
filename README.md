# Azure-ML-automation-research

### `pipelineJob` Definition
```mermaid
flowchart TD
    subgraph Azure
        
        subgraph Azure ACR
            B(DockerEnv)
        end

        subgraph Azure ML
                C(Compute)
                subgraph Scheduling_Job
                    A(pipelineJob) 
                end 
        end 

    end 

    subgraph GitHub Action
        D(azure-ml-setup.yml)
    end 
    
    D --> |1|B 
    D --> |3|Scheduling_Job
    D --> |2|C
    C --> A
    B --> A
    A --> |every month|A
 
    
    linkStyle 0,1,2,3,4,5 stroke-width:.3px;
```

### Tasks of `pipelineJob` in details
```mermaid
flowchart TD
    subgraph pipelineJob
        A[prepare_dataset]
        B[setup_automl]
        C[monitor_automl]
        D[register_automl] 
        E[publish_endpoint]
    end 

    F(datastore/iris mltable)
    G(model training... for about 12 mins.) 
    
    A --> B 
    C --> D 
    D --> E 
    A --> F 
    B --> G     
    G <--Completed---> C
    F --> G
```
<details>
  <summary>Result view of pipeline</summary>
  <img src="https://github.com/user-attachments/assets/edb2b43d-123e-4df0-8faf-a4b6fbe50bc6"></img>
</details>
