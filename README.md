# mlflow

Experiments with mlflow

## Create python virtual environment

Setup the python virtial environment including `mlflow` and dependencies.

```
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

Runs the tracking server on [http://127.0.0.1:5000/](http://127.0.0.1:5000/). The tracking server is configured to maintain tracking information at `./store/backend` and artifacts at `./store/artifacts` - thus simulating the ability to separate the persistence of these stores - for example, to maintain the artifacts in S3 object storage.

```
./tracking-server
```

### Run the simple example

The simple example demonstrates registration of execution metrics and artifacts to the tracking server.

```
./tracking-example/run
```

See the [mlflow web ui of the tracking server](http://127.0.0.1:5000/#/experiments/0) for the outcome of the run.

### Specify the 'target' tracking server

When the code runs it needs the URI of the tracking server - which can be specified in the following ways...

#### In code

See example in `tracking-example/mlflow_tracking.py`...

```
remote_server_uri = "http://127.0.0.1:5000"
mlflow.set_tracking_uri(remote_server_uri)
```

#### Environment Variable

See example in `tutorial/sklearn_elasticnet_wine/train`...

```
export MLFLOW_TRACKING_URI="http://127.0.0.1:5000"
python train.py
```

or

```
MLFLOW_TRACKING_URI="http://127.0.0.1:5000" python train.py
```
