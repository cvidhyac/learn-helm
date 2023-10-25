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

### Helm pipeline functions

- We can chain functions in helm for simple value transformations - eg. convert to upper/camel/title
  case
- Use the `|` character
- Multiple functions can be chained.

Eg. `password : {{- .Values.password | b64 | quote }}`

Will remove trailing new lines from a password, convert to base64 with Quotes.

### Helm Default values

Chain a Value to be substituted using `default` function.

Eg: `image: {{- .Values.image.name }}:{{- .Values.image.tag | default Chart.appVersion }}`

In Chart.yaml, we specify

```yaml
appVersion: alpine-2.4
```

And in values.yaml we specify

```yaml
image.name: nginx
image.tag: ""
```

Then final value substituted is `image: nginx:alpine-2.4`

### Helm Conditionals

Like other scripting languages, helm supports `if`, `else` and `else if` blocks.

For example, if a label should be included only if a flag is set, then we can use a conditional
block to derive the value of the label.

Also, we can even skip creating kubernetes resources based on flags.

ex:

serviceaccount.yaml - will be created only if the values.yaml specifies `serviceAccount.create = true`
```shell
{{- .Values.serviceAccount.create }}

apiVersion: v1
kind: ServiceAccount
metadata:
name: {{ .Release.name }}-robot-ca

{{ end }}

```

```shell
apiVersion: apps/v1
kind: Deployment
metadata:
    {{- if empty .Values.orgLabel}}
    labels:
      name: {{- Values.orgLabel.default }}
    {{- else if eq .Values.orgLabel "hr" }}
    labels:
      name: "human resources"
    {{- else if eq .Values.orgLabel "payroll" }}
    labels:
      name: "payroll"
    {{else }}
    labels:
      name: "unknown"
    {{- end }}
```
Example values.yaml
```yaml
serviceAccount:
  create: true
  orgLabel: hr
```
### Helm Scopes

- `.` character helps navigate scope hierarchy. Default is root scope.
- The `with` keyword helps provide shortcuts to the scope hierarchy where it could be very deep.
  - Eg. to access `.Values.app.ui.fg.name`, `.Values.app.ui.db.name`, enclose them in a `with` block.
- Use `$.` to access a root scoped variable from within a `with` scoped block.

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{- .Release.name}}-configmap
data:
  {{- with .Values.app }}
    {{- with .ui }}
      foreground: {{ .fg.name }}
    {{- end }}
    {{- with .db }}
      databaseName: {{ .db.name }}
      releaseName: {{ $.Release.Name }}
    {{- end }}
  {{- end }}
```

### Helm Ranges

- Useful to populate a list block in a template. Eg. list of countries in a configmap.

Example to apply the `range` keyword

```shell
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    name: {{ .Release.Name }}-configmap
data:
  regions:
  {{- range .Values.regions }}
    {{ . }}
  {{- end }}
```

Values.yaml for this example:

```yaml
appVersion: v1
regions:
  - ohio
  - new york
  - florida
  - georgia
```