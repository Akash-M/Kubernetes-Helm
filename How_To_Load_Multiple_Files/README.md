# Kubernetes + Helm

## Loading multiple files :

1: Create a charts folder in the root directory of your project and create two folders inside it
    -  One folder to store the files which need to be attached to the volume (We will name it as ```files``` for now.)
    -  One folder to store the helm charts (We will name it as ```templates```)

2: Create the base helm chart (name it as ```Chart.yaml``` inside the charts folder). The contents will be as follows:

     ```
     apiVersion: v1
    appVersion: "1.0"
    description: A Helm chart for Kubernetes
    name: sample_project
    version: 0.1.0
    ```

 3: Now create a ```configmap.yaml``` file inside the templates folder which contains the helm chart as follows:

    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: "sample_project"
      labels:
      app: "sample_project.name"
      chart: "sample_project.chart"
     ```

Your folder structure would look something like this:
  ```
> base-dir
  > charts
    > files
     - example1.json
     - example2.json
  > templates
    - configmap.yaml
  - Chart.yaml
  ```

  4: Create some sample files (can be of any type – in this case we create some json files) inside the ‘files‘ folder and then define the data property in the config map to load all the files present in the files folder.

    ```
    apiVersion: v1
    kind: ConfigMap
    metadata:
      name: "sample_project"
      labels:
      app: "sample_project.name"
      chart: "sample_project.chart"
    data:
     {{- $root := . -}}
      {{- range $path, $bytes := .Files.Glob "files/*.json" }}
      {{ base $path }}: |-
    {{ $root.Files.Get $path | indent 4}}
     {{- end }}
     ```

The Glob type allows us to read multiple files as mentioned in the helm [documentation](https://github.com/helm/helm/blob/master/docs/chart_template_guide/accessing_files.md)

  5: Run the command to check if it loads the data from the json files correctly
```helm install charts/ --dry-run --debug```.
You should be able to see the data from the json files loaded in the output as follows:
```
NAME:   mouthy-seagull
REVISION: 1
RELEASED: Tue Nov 27 10:38:18 2018
CHART: sample_project-0.1.0
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
{}

HOOKS:
MANIFEST:

Source: sample_project/templates/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: "sample_project"
  labels:
    app: "sample_project.name"
    chart: "sample_project.chart"
data:
  example1.json: |-
    {
      "name": "example1.json",
      "details": "This is an example json file"
    }

  example2.json: |-
    {
      "name": "example2.json",
      "details": "This is another example json file"
    }
   ```

  The code can be checked out from the [source-code](https://github.com/Akash-M/Kubernetes-Helm/tree/master/Loading_Multiple_Files/source-code/charts) folder
