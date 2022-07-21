# mlflow

Experiments with mlflow

- [mlflow](#mlflow)
  - [Create python virtual environment](#create-python-virtual-environment)
  - [Concepts](#concepts)
  - [Remote Tracking Server](#remote-tracking-server)
    - [Run the 'remote' Tracking Server](#run-the-remote-tracking-server)
    - [Run the simple example](#run-the-simple-example)
    - [Specify the 'target' tracking server](#specify-the-target-tracking-server)
      - [In code](#in-code)
      - [Environment Variable](#environment-variable)
  - [Tutorial](#tutorial)
    - [sklearn_elasticnet_wine](#sklearn_elasticnet_wine)
      - [Run the model training](#run-the-model-training)
      - [Run the model as a project](#run-the-model-as-a-project)
      - [Serve the model](#serve-the-model)

## Create python virtual environment

Setup the python virtial environment including `mlflow` and dependencies.

```bash
./create-venv
```

## Concepts

See docs - [https://www.mlflow.org/docs/latest/concepts.html](https://www.mlflow.org/docs/latest/concepts.html).

Four components:
* **Tracking** - API and UI for logging (parameters, versions, metrics, artifacts, etc.)
* **Projects** - standard packaging for reusable data science
* **Models** - convention for packaging machine learning models
* **Model Registry** - centralised model store, API and UI for collaborative full lifecycle model management

## Remote Tracking Server

For docs see here - [https://www.mlflow.org/docs/latest/tracking.html](https://www.mlflow.org/docs/latest/tracking.html).

### Run the 'remote' Tracking Server

Runs the tracking server on [http://127.0.0.1:5000/](http://127.0.0.1:5000/). The tracking server is configured to maintain tracking information at `./store/backend` and artifacts at `./store/artifacts` - thus simulating the ability to separate the persistence of these stores - for example, to maintain the artifacts in S3 object storage. The tracking information is maintained in a `sqlite` database `store/backend/mlflow.db` (rather than simple filesystem), since a database is required to support registration of models.

```bash
./tracking-server
```

### Run the simple example

The simple example demonstrates registration of execution metrics and artifacts to the tracking server.

```bash
./tracking-example/run
```

See the [mlflow web ui of the tracking server](http://127.0.0.1:5000/#/experiments/0) for the outcome of the run.

### Specify the 'target' tracking server

When the code runs it needs the URI of the tracking server - which can be specified in the following ways...

#### In code

See example in `tracking-example/mlflow_tracking.py`...

```python
remote_server_uri = "http://127.0.0.1:5000"
mlflow.set_tracking_uri(remote_server_uri)
```

#### Environment Variable

See example in `tutorial/sklearn_elasticnet_wine/train`...

```bash
export MLFLOW_TRACKING_URI="http://127.0.0.1:5000"
python train.py
```

or

```bash
MLFLOW_TRACKING_URI="http://127.0.0.1:5000" python train.py
```

## Tutorial

Ensure the tracking server is running...

```bash
./tracking-server
```

Check the UI at [http://127.0.0.1:5000/](http://127.0.0.1:5000/).

### sklearn_elasticnet_wine

#### Run the model training

```bash
$ ./tutorial/sklearn_elasticnet_wine/train 
```

_(output)_
```
Elasticnet model (alpha=0.500000, l1_ratio=0.500000):
  RMSE: 0.7931640229276851
  MAE: 0.6271946374319586
  R2: 0.10862644997792614
Successfully registered model 'ElasticnetWineModel'.
2022/07/21 11:06:08 INFO mlflow.tracking._model_registry.client: Waiting up to 300 seconds for model version to finish creation.                     Model name: ElasticnetWineModel, version 1
Created version '1' of model 'ElasticnetWineModel'.
```

In addition to the runs ([Experiments](http://127.0.0.1:5000/#/experiments/0)) the registered model can be visualised at [http://127.0.0.1:5000/#/models](http://127.0.0.1:5000/#/models).

Run the model with different parameterisation...

```bash
./tutorial/sklearn_elasticnet_wine/train <alpha> <l1_ratio>
```

e.g.
```bash
./tutorial/sklearn_elasticnet_wine/train 0.3 0.7
```

The model resulting from each run is [registered as a new version](http://127.0.0.1:5000/#/models/ElasticnetWineModel). The runs can be compared on the [Experiments tab](http://127.0.0.1:5000/#/experiments/0).

#### Run the model as a project

The file `tutorial/sklearn_elasticnet_wine/MLproject` specifies the mlflow project, which specifies the parameters, their default values, etc...

```yaml
name: tutorial

conda_env: conda.yaml

entry_points:
  main:
    parameters:
      alpha: {type: float, default: 0.5}
      l1_ratio: {type: float, default: 0.1}
    command: "python train.py {alpha} {l1_ratio}"
```

...along with the environment specifcation in the file `tutorial/sklearn_elasticnet_wine/conda.yaml`...

```yaml
name: tutorial
channels:
  - conda-forge
dependencies:
  - python=3.7
  - pip
  - pip:
      - scikit-learn==0.23.2
      - mlflow>=1.0
      - pandas
```

The project is run with the command `mlflow run <project-uri> <args>` - which can be invoked with...

```bash
./tutorial/sklearn_elasticnet_wine/run-project
```

...or with args to set parameters...

```bash
./tutorial/sklearn_elasticnet_wine/run-project -P alpha=0.3 -P l1_ratio=0.7
```

#### Serve the model

Amongst the artifacts of a run are two files:
* MLmodel<br>
  _A metadata file that tells MLflow how to load the model_
* model.pkl<br>
  _A serialized version of the linear regression model that was trained_

These were created as a result of the following python call `mlflow.sklearn.log_model()` to save the model...

```python
# Register the model
# There are other ways to use the Model Registry, which depends on the use case,
# please refer to the doc for more information:
# https://mlflow.org/docs/latest/model-registry.html#api-workflow
mlflow.sklearn.log_model(lr, "model", registered_model_name="ElasticnetWineModel")
```

From the Experiments tab of the mlflow web UI the run can be inspected to obtain the full path to the model, for example - `mlflow-artifacts:/0/23ca8ae6e5c74b85b95a4cb6bf64634c/artifacts/model`.

With this path the model can be served at [http://127.0.0.1:1234](http://127.0.0.1:1234)...

```bash
export MLFLOW_TRACKING_URI="http://127.0.0.1:5000"
mlflow models serve -m mlflow-artifacts:/0/23ca8ae6e5c74b85b95a4cb6bf64634c/artifacts/model -p 1234
```

Once running as a service, the model can be interrogated for predictions against given input data...

```bash
curl -X POST -H "Content-Type:application/json; format=pandas-split" --data '{"columns":["alcohol", "chlorides", "citric acid", "density", "fixed acidity", "free sulfur dioxide", "pH", "residual sugar", "sulphates", "total sulfur dioxide", "volatile acidity"],"data":[[12.8, 0.029, 0.48, 0.98, 6.2, 29, 3.33, 1.2, 0.39, 75, 0.66]]}' http://127.0.0.1:1234/invocations
```

_(output)_
```
[4.662849784340928]
```
