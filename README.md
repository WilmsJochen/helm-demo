# Helm demo 
This repo contains helm chart to demo the use case of helm and showcase some handy features in the charts.

## Environment
You can execute this exercise on all machines, but if you do not have the helm binary installed, you could use the playgrounds of following website.
https://killercoda.com/playgrounds/scenario/kubernetes

## Instructions

### Checkout Git repo
To use our code, we need to clone this git repo.

1) Clone the git repo
 ```
 git clone https://github.com/WilmsJochen/helm-demo.git
 ```
2) Change your directory to the repo
 ```
 cd helm-demo
 ```
3) Have a look at the folder and subfolders
```
 ls ./
```

### template the server chart.
Before we install or upgrade charts into the cluster we have to explain how helm works.
Templating a helm chart can be handy to link all the helm files.

1) list all the templates in the the templates folder and investigate the configmap.yaml file.
```
cd server
ls ./templates
cat configmap.yaml
```

2) Link the configmap template with variables in the values.yaml file.
```
cat values.yaml
```
Look at the variable set at `config.message`

3) template only the confimap.
```
helm template -s templates/configmap.yaml . 
```
4) change the message to a custom message in the values.yaml file
```
nano values.yaml
```
5) Check the output of this change
```
helm template -s templates/configmap.yaml . 
```
6) finally template the full chart and browse through all the generated files.
```
helm template . --output-dir generated 
ls ./generated/server/templates/
```

### Install/upgrade server chart
The previously templated kubernetes resources can be installed in the cluster in one go. 
This will make sure your resources will be grouped by a helm chart.

1) Validate the chart
```
helm lint ./
helm show all ./
helm install server . --dry-run
```

2) Install the chart
```
cd server
helm dep update
helm install server ./
```

3) validate helm installation
```
helm list
kubectl get pods
kubectl get deployments
kubectl get configmap
kubectl get service
```
4) Change message in the values.yaml and upgrade the cluster.
```
nano values.yaml
helm upgrade server ./
kubectl get configmap server-server -oyaml
```

5) Package and push chart to the dockerhub registry.
```
helm package .
helm push server-0.0.1.tgz  oci://registry-1.docker.io/desyco
```

### Pull and install job chart.
1) install a publicly available chart. 
```
helm install job oci://registry-1.docker.io/desyco/job
```
2) upgrade chart with values.yaml change
```
k delete job job-job
helm upgrade job oci://registry-1.docker.io/desyco/job --set config.url=http://server-server
```

3) pull the helm chart locally for adaptations.
```
k delete job job-job
helm pull oci://registry-1.docker.io/desyco/job --version 0.0.1 --untar --untardir job-download
cd job-download/job
nano values.yaml
helm upgrade job ./
```

### Install environment chart.
The environment chart has dependencies on helm charts of the server and the job. Since we have dependencies, we also need to keep these in sync.

1) Have a look at the chart.yaml file
```
cat Chart.yaml
```

2) Update dependencies of project charts.
```
helm dependency update
```

3) Adapt config of both projects in the values.yaml
```
nano ./values.yaml
```

4) Install the environment chart in the cluster.
```
helm dependency update
helm install env .
kubectl get pods
kubectl logs <JOB_POD_NAME>
```

5) Change the message and update the chart
```
nano values.yaml
helm dependency update
helm upgrade env .
```

### Create a new chart and add to the environment chart
Create a new folder with your own custom chart. You can take some inspiration form the other charts available.
Next add this new chart in the dependencies of the environment and deploy everything at once.
