# learn-helm

## Practice

I used the [free playground here](https://killercoda.com/playgrounds/scenario/ubuntu)

In the playground, use the instructions for installing
helm : [helm docs](https://helm.sh/docs/intro/install/)

`brew install helm` is also useful for local practice.

## Useful Helm commands

| Usage                                     | Command/Example                                                    |
|-------------------------------------------|--------------------------------------------------------------------|
| Helm version                              | `helm version`                                                     |
| Helm environment                          | `helm env`                                                         |
| Add a repository to cluster               | `helm repo add bitnami https://charts.bitnami.com/bitnami`         |
| List repositories                         | `helm repo list`                                                   |
| Install a release                         | `helm install dazzling-web bitnami/nginx`                          |
| Install release with overrides            | `helm install dazzling-web bitnami/nginx --set go-version = 1.24 ` |
| Install release with many overrides       | `helm install dazzling-web bitnami/nginx --values=overrides.yaml`  |
| Uninstall a release                       | `helm install dazzling-web`                                        |
| List the releases                         | `helm list`                                                        |
| Search in Artifact hub                    | `helm search hub nginx`                                            |
| Search in repo                            | `helm search repo nginx`                                           |
| Upgrade a release                         | `helm upgrade dazzling-web bitnami/nginx --version=13.2.3`         |
| Lint a chart                              | `helm lint ./nginx_chart`                                          |
| Verify a chart with replaced placeholders | `helm template ./nginx-chart`                                      |
| Dry run for chart                         | `helm install dazzling-web bitnami/nginx --dry-run`                |


### Creating a custom chart with structure

`helm create webapp-nginx`

Refer - [webapp-nginx](./webapp-nginx) for generated files with this command
