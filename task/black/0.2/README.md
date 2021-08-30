# Black (Python Code Prettier)

This task can be used to format the python source code using [Black](https://github.com/psf/black) which is an Opinionate Code Formatter.

## Installing the Task

```bash
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/black/0.2/black.yaml
```

## Parameters

- **image**: The container image to use. (_Default_: `docker.io/cytopia/black:latest-0.2@sha256:2ec766f1c7e42e6b59c0873ce066fa0a2aa2bf8a80dbc1c40f1566bb539303e0`)
- **requirements_file**: The name of the requirements file inside the workspace that will be used by pip to install black.
      If this file does not exist the task will not fail, but pip will not install anything. (_Default_: `requirements.txt`)
- **args**: The extra params along with the file path needs to be provided as the part of `args`. (_Default_: `["--help"]`)

## Workspaces

- **shared-workspace**: The workspace containing python source code which we want to format. It can be a shared workspace with the `git-clone` task or a `ConfigMap` mounted containing some files.

## Usage

1. Create the `git-clone` task

```bash
kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.1/git-clone.yaml
```

2. Create the PVC
3. Apply the required tasks

4. Create the Pipeline and PipelineRun for `Black`(Python Code Formatter)

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: python-formatter-pipeline
spec:
  workspaces:
    - name: shared-workspace
  tasks:
    - name: fetch-repository
      taskRef:
        name: git-clone
      workspaces:
        - name: output
          workspace: shared-workspace
      params:
        - name: url
          value: https://github.com/wumaxd/pylint-pytest-example
        - name: subdirectory
          value: ""
        - name: deleteExisting
          value: "true"
    - name: python-black-run #python code prettier
      taskRef:
        name: black
      runAfter:
        - fetch-repository
      workspaces:
        - name: shared-workspace
          workspace: shared-workspace
      params:
        - name: args
          value: [".", "--check", "--diff"]
        - name: image
          value: python:3.8-slim
---
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: python-formatter-pipeline-run
spec:
  pipelineRef:
    name: python-formatter-pipeline
  workspaces:
    - name: shared-workspace
      persistentvolumeclaim:
        claimName: black-python-pvc
```