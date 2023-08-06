# Kubernetes Helm Guide

## Table of Contents

- [Chapter 1: Helm Basics](#chapter-1-helm-basics)
- [Chapter 2: Developing Helm Charts](#chapter-2-developing-helm-charts)  
- [Chapter 3: Chart Repositories](#chapter-3-chart-repositories)
- [Chapter 4: Advanced Chart Usage](#chapter-4-advanced-chart-usage)
- [Chapter 5: Chart Testing, Validation, and CI/CD](#chapter-5-chart-testing-validation-and-cicd)
- [Chapter 6: Securing Helm](#chapter-6-securing-helm)
- [Chapter 7: Troubleshooting Helm](#chapter-7-troubleshooting-helm) 
- [Chapter 8: Conclusion](#chapter-8-conclusion)

## Chapter 1: Helm Basics

Helm is a package manager for Kubernetes that helps you install and manage applications and resources on your Kubernetes cluster. At its core, Helm uses a packaging format called charts. A chart is a collection of Kubernetes manifest files bundled together.

Charts simplify deployment by letting you package Kubernetes manifests, configurations, and templates into a reusable bundle. This chapter will cover the basic concepts and usage of Helm charts.

### What is a Helm Chart?

A Helm chart can be thought of as a package that contains:

- Kubernetes resource manifests - Deployments, Services, ConfigMaps etc
- Configuration templates - customizable YAML files
- Default configuration values
- Dependencies - charts that this chart requires
- Documentation - README, notes, etc

For example, a MySQL chart would contain Deployments and Services for the MySQL server and client. It would have configurable values like the MySQL server root password, the storage size, the node selector, and so on.

The chart abstracts away the specifics into the values.yaml file while keeping the template files very generic. This allows the MySQL chart to be deployed multiple times with different configurations.

### Anatomy of a Chart

A Helm chart contains the following folder structure and files:

```
mychart/
  Chart.yaml          # Contains metadata
  values.yaml         # Default config values
  charts/             # Charts that this chart depends on
  templates/          # Template YAML manifests
  templates/NOTES.txt # Usage notes
```

The core components are:

- Chart.yaml - Contains metadata like the name, version, appVersion and dependencies. Required.

- values.yaml - Specifies the default configuration values. This is the main way to customize the application when deploying. Optional.

- templates/ - Holds the Kubernetes manifest templates. Names need to follow naming conventions. Required.

- charts/ - Holds dependency charts upon which this chart depends. Optional.

There are also a few other optional files like templates/NOTES.txt which contains usage instructions printed after installation.

To summarize, Helm charts bundle together all the configurable Kubernetes resources needed to run an application as a single deployable package.

### Installing Helm

Before we can start using charts, we need to install the Helm client on our local machine.

Here are the steps to install Helm:

1. Download the latest Helm release binary from https://github.com/helm/helm/releases

2. Unzip the file and copy the binary to a directory in your PATH, e.g.

```
$ curl -L https://git.io/get_helm.sh | bash
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   617  100   617    0     0   2416      0 --:--:-- --:--:-- --:--:--  2416  
100 11.8M  100 11.8M    0     0  4083k      0  0:00:02  0:00:02 --:--:-- 4073k
$ sudo cp linux-amd64/helm /usr/local/bin/helm
```

3. Run `helm version` to verify it installed correctly:

```
$ helm version  
version.BuildInfo{Version:"v3.1.2", GitCommit:"d878d4d45863e42fd5cff6743294a11d28a9abce", GitTreeState:"clean", GoVersion:"go1.13.8"}
```

With Helm installed, you can now start using it to manage charts on your Kubernetes cluster.

### Deploying Your First Chart

Let's get started with Helm by deploying a simple chart available on the official charts repository.

The Nginx chart creates a basic Nginx webserver pod using Kubernetes Deployments and Services.

Here are the steps:

1. Add the Helm charts repository:

```
$ helm repo add stable https://charts.helm.sh/stable 
```

2. Update the repo to fetch the latest charts:

```
$ helm repo update
```

3. Search for charts:

```  
$ helm search repo nginx
NAME                    CHART VERSION   APP VERSION     DESCRIPTION
stable/nginx-ingress    1.36.0          0.32.0          An nginx Ingress controller that uses ConfigMap...   
stable/nginx            1.12.0          1.16.0          Chart for the nginx server
```

4. Install the nginx chart:

```
$ helm install my-nginx stable/nginx
NAME: my-nginx
LAST DEPLOYED: Tue Aug 6 14:20:42 2019
NAMESPACE: default  
STATUS: deployed
REVISION: 1
TEST SUITE: None
```

This will pull the nginx chart from the repo and deploy the Kubernetes resources defined in it.

We now have a simple nginx deployment running on our cluster. We can upgrade, rollback or delete it using helm commands. 

This covers the basic concepts of Helm charts and shows how you can easily install pre-built charts. In the next chapter we will look at how to develop your own charts.

## Chapter 2: Developing Helm Charts

In the previous chapter we installed and used pre-existing Helm charts. Now let's look at how to create our own charts to package and deploy applications.

Developing charts involves creating the chart files with Kubernetes manifests, configurations, and templates. This chapter will cover best practices for structuring chart files and directories.

### Chart Structure Best Practices

Though Helm charts can be organized different ways, here are some standard recommendations:

- Place all Kubernetes manifests for the application in the templates/ folder

- Create a template file for each resource kind (Deployment, Service, etc)

- Use values.yaml for default configuration values

- Make heavy use of templating to parameterize the charts

- Keep charts modular and reusable across environments

- Follow naming conventions for files and templates

For example, a chart for a web app might look like:

```
mychart/
  Chart.yaml
  values.yaml
  charts/ 
  templates/
    deployment.yaml
    service.yaml
    ingress.yaml
  templates/NOTES.txt
```

This keeps each major manifest separate for easy templating and overriding.

### The Chart.yaml File

The Chart.yaml contains metadata about the chart such as name, version and description.

Here is an example Chart.yaml:

```yaml
apiVersion: v2
name: mychart
description: A Helm chart for my application  
type: application
version: 0.1.0
appVersion: 1.0
dependencies:
  - name: nginx
    version: 1.2.0 
    repository: https://charts.bitnami.com/bitnami
```

The key fields are:

- name - The name of the chart
- version - The semantic version of the chart 
- appVersion - The version of the packaged application
- dependencies - Other charts upon which this chart depends

### Templates

Templates allow you to parameterize the Kubernetes manifests. This lets you customize deployments without changing the template files. 

Templates use double curly braces {{ }} to inject values. They also provide control logic like loops, conditionals and variables.

For example, a basic nginx Deployment template:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-nginx
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels: 
        app: nginx
    spec:
      containers:
        - name: nginx
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

### Values Files

The values.yaml file contains the default configuration values for the chart. 

For example:

```yaml
# values.yaml

replicaCount: 1

image:
  repository: nginx
  tag: stable
   
service:
  type: ClusterIP
  port: 80
```

The values file is flexible and allows you to use nested values. 

During deployment, custom values can be passed with helm install -f myvalues.yaml to override defaults.

Together, templates and values files provide powerful parameterization and customization of Kubernetes manifests for each deployment.

### Conclusion

In this chapter we looked at best practices for organizing your own Helm charts. Keeping things modular with good separation of concerns will allow your charts to be robust, reusable and maintainable for different environments and customizations.

In the next chapter we will look at sharing charts using chart repositories.

## Chapter 3: Chart Repositories

In the previous chapters we installed charts from the official Helm charts repository and created our own example charts. In this chapter we will look at hosting and managing custom chart repositories.

Chart repositories are a way to package and distribute Helm charts. Some key benefits include:

- Share charts with others
- Publish new chart versions
- Install charts without downloading/installing manually 
- Manage chart dependencies

### Hosting a Chart Repository

There are a few options for hosting your own Helm chart repository:

- Use a static web server to serve an `index.yaml` file along with packaged charts
- Host your own ChartMuseum instance
- Use a storage bucket in S3/GCS/Azure/GitHub Pages

For example, we can use ChartMuseum to host charts:

```bash
# Install ChartMuseum
$ helm install chartmuseum stable/chartmuseum

# Package chart and copy to ChartMuseum storage  
$ helm package mychart/
$ curl --data-binary "@mychart-0.1.0.tgz" http://chartmuseum.default.example.com/api/charts 

# Fetch index file from ChartMuseum to get chart listing
$ curl http://chartmuseum.default.example.com/index.yaml
```

Now others can add your custom repo and install charts from it!

### Adding Chart Repositories

To use a custom chart repository, you first need to add it to your Helm config:

```
$ helm repo add myrepo http://chartmuseum.default.example.com
```

This adds a new repo named `myrepo` pointing to the given URL. 

You can view configured repos:

```
$ helm repo list
NAME    URL
stable  https://charts.helm.sh/stable
myrepo  http://chartmuseum.default.example.com
```

To update the repo indexes after adding charts, run: 

```
$ helm repo update
```

Now you can search and install charts from this repository.

### Managing Dependencies 

Charts can depend on other charts. For example, a web application might depend on a database chart.

These dependencies are defined in Chart.yaml:

```yaml 
dependencies:
- name: mysql
  version: 6.2.0
  repository: http://myrepo.com
```

With dependencies configured, `helm install` will automatically download and install the dependent charts from the defined repos.

This allows you easily install complex multi-service applications.

In this chapter we looked at hosting and sharing Helm charts using custom repositories. We also covered managing chart dependencies.

In the next chapter we will dig deeper into the powerful templating and logic Helm provides.

## Chapter 4: Advanced Chart Usage 

In previous chapters we covered the basics of creating, packaging and sharing Helm charts. Now let's look at some more advanced Helm features for writing robust, flexible and reusable charts.

### Templating Functions

Helm provides many built-in template functions that make it easier to write charts. Some commonly used ones include:

- .Values - Access values passed during install
- .Release - Info about the release
- .Chart - Metadata about the chart 
- .Files - Access files bundled in the chart
- .Capabilities - Kubernetes cluster info   

For example:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-config 
data:
  myvalue: {{ .Values.myvalue }}
```

### Logic Statements

Control structures like if/else statements, with/range loops and defines can be added:

```yaml 
{{- if .Values.rbacEnabled }}
# Role bindings
{{- end }}

{{- range .Values.nodes }}  
# Loop to create one pod per node
{{- end }}

{{- define "mychart.app" -}}
  labels: 
    app: {{ .Chart.Name }}  
{{- end }}
```

This provides full programmatic control for flow and logic.

### Complex Values

The values passed during install can use nested objects, lists and variables:

```yaml
servers:
  - name: server1  
    url: https://server1.example.com

plugins:
  - plugin1
  - plugin2
```

Values can also be scoped to only be accessible to certain templates.

### Writing Helpers

You can write custom template functions called helpers using the `helm.sh/helm/v3/pkg/chart` module:

```yaml 
{{- define "mychart.app" -}}
  labels:
    app: {{ .Chart.Name }}
{{- end }}
```

Then access it in templates:

```yaml
metadata: 
{{ include "mychart.app" . | indent 2 }}
```

This keeps things DRY (Don't Repeat Yourself).

### Overriding Resources

Sometimes you want to override just part of a resource manifest. This can be done using the `helm.sh/helm/v3/pkg/action` module. 

For example, to patch an existing Secret:

```yaml
{{- $original := get .Original "Secret" "mysecret" -}}
{{- $modified := omit .Modified "data" -}}
{{- $target := mustMerge $original $modified -}}
{{- $target | toYaml | nindent 2 }}
```

This allows surgically modifying resources without needing to redefine the entire object.

### Conclusion

In this chapter we covered some advanced Helm features like templating functions, control logic, helpers, and overriding resources. Using these features allows creating truly robust and reusable charts.

Let me know if you would like me to expand on any of these topics or add additional advanced templating examples.

## Chapter 5: Chart Testing, Validation, and CI/CD

As with any code, it is important to test Helm charts to catch issues early and ensure proper behavior. In this chapter we will look at ways to validate charts. We will also discuss automating chart building and release through CI/CD pipelines.

### Linting and Testing

Helm provides a linting tool that checks charts for formatting issues and other common problems:

```bash
# Lint a chart
$ helm lint mychart

==> Linting mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

This validates the chart structure without needing to install it.

For validation testing, the `helm/chart-testing` tool is highly recommended:

```bash
# Install chart-testing 
$ helm plugin install https://github.com/helm/chart-testing

# Run tests on a chart
$ ct lint-and-install --config ct.yaml
```

The ct.yaml file defines your test configurations and clusters. This allows thoroughly testing a chart before release.

### Validating Changes 

To validate chart changes don't introduce regressions, you can diff the manifests between revisions:

```bash
$ helm template mychart
# Save output to manifest-v1.yaml

$ helm template mychart --version 0.2.0
# Save output to manifest-v2.yaml 

$ diff manifest-v1.yaml manifest-v2.yaml 
```

This lets you compare deployed resources for unintended changes when upgrading charts.

### Automating with CI/CD

Chart development and release can be automated using CI/CD tools like GitHub Actions:

```yaml
on: push 

jobs:

  lint-test:
    runs-on: ubuntu-latest  
    steps:
    - uses: actions/checkout@v2
    - run: helm lint mychart
    - run: ct install --config ct.yaml

  release:
    needs: lint-test
    runs-on: ubuntu-latest  
    steps:
    - run: helm package mychart 
    - run: helm repo update
    - run: helm upgrade --install myrelease mychart
```

This automatically builds and deploys your chart on each commit or merge to the main branch. 

### Conclusion

Proper validation through linting, testing and CI/CD automation is crucial for developing robust Helm charts. This helps catch issues early and simplify the chart release process.

Let me know if you would like me to expand on any section of this chapter. I can add more examples of testing and CI workflows.

## Chapter 6: Securing Helm 

As with any Kubernetes deployment, it is important to follow security best practices when using Helm. In this chapter we will look at ways to create secure and permissions-controlled chart deployments.

### Chart Values Security

Helm chart values can potentially contain sensitive data like passwords and keys. Avoid putting secrets directly into values.yaml. Instead, use Secret resource manifests and inject the secret name.

For example:

```yaml
# Create Kubernetes Secret
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: {{ .Values.password | b64enc }}

# Reference secret in Deployment
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mysecret
        key: password
```

This keeps passwords and keys out of the charts, even if the chart code itself is public.

### Signed Charts 

Helm has an experimental feature for cryptographically signing charts to ensure integrity. This can prevent chart tampering or spoofing.

```bash
# Generate signing key pair 
$ openssl genrsa -out keypair.pem 2048

# Sign a chart package
$ helm package --sign --key ./keypair.pem --keyring ./keyring.gpg mychart

# Verify signed chart
$ helm verify ./mychart-0.1.0.tgz 
```

While signatures are not yet common practice, they provide chart source verification.

### RBAC Permissions 

By default, Helm installs have cluster-admin privileges. This can be restricted using Kubernetes RBAC.

Create a ServiceAccount for the Tiller pod:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller-deploy
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: tiller-deploy
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller-deploy
    namespace: kube-system
```

Then install Tiller using the ServiceAccount:

```bash
$ helm init --service-account tiller-deploy
```

Now chart deployments will run with defined RBAC rules.

### Conclusion

Proper chart security involves avoiding secrets in values, enabling source verification, and limiting RBAC permissions. This helps reduce the risk of charts being tampered with or misused.

Let me know if you would like me to expand on any section of this chapter. I can add more examples of securing Helm workflows.

## Chapter 7: Troubleshooting Helm

As with any complex deployment system, you may occasionally run into issues when using Helm. This chapter covers some common problems and how to troubleshoot them.

### Investigating Installation Issues

If a chart fails to install or upgrade, your first step should be checking the installation logs:

```bash
# View installation logs
$ helm install --debug --dry-run mychart 

# Get logs for a release
$ helm status my-release
```

The logs will contain events and pod statuses useful for identifying root causes like invalid manifests, resource conflicts, insufficient RBAC permissions etc.

You can also inspect the Kubernetes resources created from the chart templates for issues:

```bash
# Render chart templates to YAML
$ helm template mychart > manifests.yaml

# Inspect the resources in manifests.yaml
$ kubectl apply --dry-run -f manifests.yaml
```

This allows parsing the rendered templates before actual installation.

### Performing Rollbacks

If a chart update fails or introduces issues, you may need to rollback the release to the previous working version:

```bash
# See release revision history 
$ helm history my-release

# Rollback to previous revision
$ helm rollback my-release 1
```

This will redeploy the old manifests and configuration for the release.

### Common Issues

Here are some common Helm issues:

- Dependency chart failed to install - Fix issues with depended chart first
- Values schema change - Use `--reuse-values` flag for compatibility  
- Invalid template rendering - Fix syntax errors in template files
- RBAC errors - Grant Tiller required permissions
- Resource conflicts - Adjust namespace/names to avoid collisions

### Conclusion

Troubleshooting charts requires inspecting logs, manifests and past revisions to identify and fix problems. Most issues boil down to chart dependency and configuration problems.

Let me know if you would like me to expand on any section of this chapter. I can add more debugging techniques and examples.

## Chapter 8: Conclusion

In this comprehensive guide, we covered everything you need to start using Helm - the package manager for Kubernetes. 

We started with an overview of Helm architecture and its core concepts of charts, repositories, releases, and values files. We then jumped into hands-on examples of:

- Installing Helm on your machine
- Deploying existing charts like Nginx 
- Developing your own charts from scratch
- Configuring chart repositories
- Writing robust charts using Helm templates
- Validating charts with linting and testing
- Automating chart releases with CI/CD
- Securing charts and cluster permissions
- Troubleshooting failed installations and rollbacks

By now you should feel comfortable packaging, deploying, and managing applications on Kubernetes using Helm. The key takeaways include:

- Helm streamlines deploying apps to Kubernetes
- Charts encapsulate all Kubernetes manifests and configs for an app
- Templating provides flexibility and customization for charts
- Repositories allow sharing and installing charts
- Follow Kubernetes best practices for security
- Test charts thoroughly to avoid issues

Helm has many additional capabilities we did not cover like plugins, scripting helpers, integration with other tools like Kustomize, and the new Helm v3 architecture. Refer to the official Helm documentation to learn more.

Thank you for following along with this hands-on guide. You now have the core knowledge to start benefiting from Helm in your Kubernetes deployments. Happy charting!

Let me know if you would like me to modify or expand this conclusion chapter in any way. Please provide your valuable feedback on the full guide as well.
