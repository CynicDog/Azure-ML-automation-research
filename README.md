# Azure-ML-automation-research


## Tasks performed within a workspace[.](https://learn.microsoft.com/en-us/azure/machine-learning/concept-workspace?view=azureml-api-2)
> For machine learning teams, the workspace is a place to organize their work. Some of the tasks you can start from a workspace include: 
> - **Create `Jobs`**: Jobs are training runs you use to build your models. You can group jobs into `experiments` to compare metrics. 
> - **Author `Pipelines`**: Pipelines are reusuable workflows for training and retrainig the model.
> - **Register `Data Assets`**: Data assets aid in management of the data you use for model training and pipeline creation.
> - **Register `Models`**: Once you have a model you want to deploy, you create a registered model. 
> - **Create online `Endpoints`**: Use a registered model and a scoring scripts to create an online endpoint. 
> 
> Besides grouping your machine learning results, workspaces also host resource configurations: 
> - `Compute targets` are used to run your experiments.
> - `Datastores` define how you and others can connect to data sources when using data assets.
> - `Security settings` cover networking, identity and access control, and encryption settings.   

### 1. Create Jobs and train models[.](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-train-model?view=azureml-api-2&tabs=azurecli)

#### Connect to the workspace
```bash
az account set --subscription <subscription ID>
az configure --defaults workspace=<Azure Machine Learning workspace name> group=<resource group>
```

#### Create a compute resource for training 
```bash
az ml compute create -n cpu-cluster --type amlcompute --min-instances 0 --max-instances 4
```

#### Submit the training job 
The az ml job create command used in this example requires a YAML job definition file.  

<details>
  <summary>View <code>job.yml</code> file</summary>
  <pre><code>$schema: https://azuremlschemas.azureedge.net/latest/commandJob.schema.json
code: src
command: >-
  python main.py 
  --iris-csv ${{inputs.iris_csv}}
  --C ${{inputs.C}}
  --kernel ${{inputs.kernel}}
  --coef0 ${{inputs.coef0}}
inputs:
  iris_csv: 
    type: uri_file
    path: wasbs://datasets@azuremlexamples.blob.core.windows.net/iris.csv
  C: 0.8
  kernel: "rbf"
  coef0: 0.1
environment: azureml://registries/azureml/environments/sklearn-1.5/labels/latest
compute: azureml:cpu-cluster
display_name: sklearn-iris-example
experiment_name: sklearn-iris-example
description: Train a scikit-learn SVM on the Iris dataset.</code></pre>
</details>

To submit the job, use the following command. The run ID (name) of the training job is stored in the `$run_id` variable:

```bash
run_id=$(az ml job create -f jobs/single-step/scikit-learn/iris/job.yml --query name -o tsv)
```

You can use the stored run ID to return information about the job. 
```
az ml job show -n $run_id --web 
```

### 2. Register the trained model[.](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-train-model?view=azureml-api-2&tabs=azurecli)

The following examples demonstrate how to register a model in your Azure Machine Learning workspace.

```bash 
az ml model create -n sklearn-iris-example -v 1 -p runs:/$run_id/model --type mlflow_model 
```


### 3. Define the endpoint[.](https://learn.microsoft.com/en-us/azure/machine-learning/how-to-deploy-online-endpoints?view=azureml-api-2&tabs=cli) 

#### Set an endpoint name 
```bash
export ENDPOINT_NAME="<YOUR_ENDPOINT_NAME>"
```

#### Configure the endpoint 
<details>
  <summary>View <code>endpoint.yml</code> file</summary>
  <pre><code>$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineEndpoint.schema.json
name: my-endpoint
auth_mode: key</code></pre>
</details>

#### Configure a deployment 
A deployment is a set of resources required for hosting the model taht does the actual inferencing. 

<details>
  <summary>View <code>blue-deployment.yml</code> file</summary>
  <pre><code>$schema: https://azuremlschemas.azureedge.net/latest/managedOnlineDeployment.schema.json
name: blue
endpoint_name: my-endpoint
model:
  path: ../../model-1/model/
code_configuration:
  code: ../../model-1/onlinescoring/
  scoring_script: score.py
environment: 
  conda_file: ../../model-1/environment/conda.yaml
  image: mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04:latest
instance_type: Standard_DS3_v2
instance_count: 1</code></pre>
</details>

#### Deploy the model locally 
First, create an endpoint with the following command. 
```bash
az ml online-endpoint create --local -n $ENDPOINT_NAME -f endpoints/online/managed/sample/endpoint.yml
```

Now, create a deployment named blue under the endpoint.
```bash
az ml online-deployment create --local -n blue --endpoint $ENDPOINT_NAME -f endpoints/online/managed/sample/blue-deployment.yml
```
