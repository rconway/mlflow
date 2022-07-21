# mlflow

Experiments with mlflow

## Create python virtual environment

Setup the python virtial environment including `mlflow` and dependencies.

```
./create-venv
```

## Run the 'remote' Tracking Server

Runs the tracking server on [http://127.0.0.1:5000/](http://127.0.0.1:5000/). The tracking server is configured to maintain tracking information and artifacts under the local directory `./tracking/mlruns`.

```
./tracking-server
```

## Run the simple example

The simple example demonstrates registration of execution metrics and artifacts to the tracking server.

```
./tracking-example/run
```

Note that the python code sets the url of the tracking server...
```
remote_server_uri = "http://127.0.0.1:5000"
mlflow.set_tracking_uri(remote_server_uri)
```

See the [mlflow web ui of the tracking server](http://127.0.0.1:5000/#/experiments/0) for the outcome of the run.
