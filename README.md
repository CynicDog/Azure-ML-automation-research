# Azure-ML-automation-research

#### Create a dedicated Service Principal for the project

```bash
    az ad sp create-for-rbac --name "{SERVICE_PRINCIPAL_NAME}" \
     --role contributor \
     --scopes /subscriptions/{SUBSCRIPTION_ID} \
     --sdk-auth
```
> This will then return a response of credentials in JSON format. Save the JSON as a repository secret with the name of `AZURE_CREDENTIALS`, which will then later be used in Azure CLI login in the GitHub workflow. 

### `pipelineJob` Definition
```mermaid
C4Context
    title  
    
    Deployment_Node(github, "GitHub") {
        Component(githubaction, "GitHub Action", "azure-ml-setup.yml", "")
    }

    Enterprise_Boundary(azure, "Azure") {
        
        Container(acr, "Acure Container Registry")

        Deployment_Node(azml, "Azure Machine Learning") {
            
            Component(compute, "Compute Cluster")
            
            Component(pipeline, "Pipeline")
            
            Component(scheduler, "Scheduler")
        }
    }

    Rel(githubaction, acr, "Linux env. with configs and data")
    UpdateRelStyle(githubaction, acr, $textColor="grey", $offsetX="-80", $offsetY="10")

    Rel(githubaction, compute, "Create compute cluster")
    UpdateRelStyle(githubaction, compute, $textColor="grey", $offsetX="-80", $offsetY="10")

    Rel(githubaction, scheduler, "Set up schedule")
    UpdateRelStyle(githubaction, scheduler, $textColor="grey", $offsetX="-80", $offsetY="10")

    Rel(acr, pipeline, "Provdes containerized Linux environment")
    UpdateRelStyle(acr, pipeline, $textColor="grey", $offsetX="-150", $offsetY="-80")

    Rel(scheduler, pipeline, "Triggers on cron event")
    UpdateRelStyle(scheduler, pipeline, $textColor="grey", $offsetX="-100", $offsetY="30")

    Rel(compute, pipeline, "Provides compute resource")
    UpdateRelStyle(compute, pipeline, $textColor="grey", $offsetX="-70", $offsetY="10")
```

### Tasks of `pipelineJob` in details
```mermaid
flowchart TD
    subgraph pipelineJob
        A[prepare_dataset]
        B[setup_automl]
        C[monitor_automl]
        D[create_endpoint] 
        E[deploy_best_model]
        G(model training... for about 12 mins.) 
    end 
    
    A --> B 
    C --> E 
    D --> E 
    B -->      
    G --Completed---> C
```
### Result view of pipeline run 
<img width="600" alt="image" src="https://github.com/user-attachments/assets/38f993f5-4c13-4a24-8995-b650579a7876">

