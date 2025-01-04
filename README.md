# learn-helm

## Practice

I used the [free playground here](https://killercoda.com/playgrounds/scenario/ubuntu)

In the playground, use the instructions for installing
helm : [helm docs](https://helm.sh/docs/intro/install/)

`brew install helm` is also useful for local practice.

## Useful Helm commands

| Usage                                        | Command/Example                                                                                   |
|----------------------------------------------|---------------------------------------------------------------------------------------------------|
| Helm version                                 | `helm version`                                                                                    |
| Helm environment                             | `helm env`                                                                                        |
| Add a repository to cluster                  | `helm repo add bitnami https://charts.bitnami.com/bitnami`                                        |
| List repositories                            | `helm repo list`                                                                                  |
| Install a release                            | `helm install dazzling-web bitnami/nginx`                                                         |
| Install release with overrides               | `helm install dazzling-web bitnami/nginx --set go-version = 1.24 `                                |
| Install release with many overrides          | `helm install dazzling-web bitnami/nginx --values=overrides.yaml`                                 |
| Uninstall a release                          | `helm install dazzling-web`                                                                       |
| List the releases                            | `helm list`                                                                                       |
| Search in Artifact hub                       | `helm search hub nginx`                                                                           |
| Search in repo                               | `helm search repo nginx`                                                                          |
| Upgrade a release                            | `helm upgrade dazzling-web bitnami/nginx --version=13.2.3`                                        |
| Lint a chart                                 | `helm lint ./nginx_chart`                                                                         |
| Verify a chart with replaced placeholders    | `helm template ./nginx-chart`                                                                     |
| Dry run for chart                            | `helm install dazzling-web bitnami/nginx --dry-run`                                               |
| Package a Helm Chart                         | `helm package --sign --key 'John smith' --keyring /Users/jsmith/.gnupg/secring.gpg ./nginx_chart` |
| Verify a helm chart downloaded from internet | `helm verify --keyring /Users/jsmith/.gnupg/secring.gpg nginx_chart-0.0.1.tgz`                    |
| Generate a helm repo index file              | `helm repo index nginx-chart-files/ --url https://example.com/charts                              |

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

serviceaccount.yaml - will be created only if the values.yaml
specifies `serviceAccount.create = true`

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
- The `with` conditional helps provide shortcuts to the scope hierarchy where it could be very deep.
    - Eg. to access `.Values.app.ui.fg.name`, `.Values.app.ui.db.name`, enclose them in a `with`
      block.
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

- Useful to populate a list/Map block in a template. Eg. list of countries in a configmap.

#### Range on a List

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

#### Range on a Map

```shell
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    {{- range $key, $value := .Values.serviceAccount.labels }}
    {{ $key }}:{{ $value | quote }}
    {{- end }}
```

```yaml
serviceAccount:
  labels:
    app: webapp
    color: pink
    kubernetes.io/managed-by: Helm
```

### Named templates (Simple)

- Name templates can be defined in `_some_name.tpl` file. The `.tpl` extension indicates helm that
  it is not a regular template file.
- With named templates imported as type `template`, we need to define them with right indentation.
- To import a template with scope variables, add a `.` next to the template.

Example template without need for scoping

```text template_without_scoping.tpl
{{- define "labels" }}
appName: webapp
appCode: ABC-001
{{- end }}
```

```text template_with_scoping.tpl
{{- define "custom" }}
app.kubernetes.io/name: {{- .Release.Name }}
{{- end }}
```

```text
{{- define "webapp-secrets" }}
Passw0rd
{{- end }}
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    { { - template "labels" } }
      { { - template "custom" . } } # Note the . character after the declaration to access scoped vars

```

```yaml
apiVersion: v1
kind: Secret
metadata:
  labels:
    name: webapp-secret
data:
  { { include "webapp-secrets" | b64 | quote } }
```

Template is an action, however include is a function. To be able to use
helm-style parameterized replacements (e.g. `{{ .Release.name }}`) in template files, we should use
the `include` directive over the `template` directive.

### Chart Hooks

Chart hooks are annotations to be added on the jobs that would perform pre-defined actions during
a helm installation or upgrade. There are specific types of helm chart hooks available.

Read - https://helm.sh/docs/topics/charts_hooks/#writing-a-hook

Hooks will execute on the specific lifecycle where they are specified. To define a hook, add the
annotation - `helm.sh/hook` on the job metadata.

Since hooks are jobs, they will require a clean-up after execution / failure.
There is another annotation called `helm.sh/hook-delete-policy` that takes care of this clean up.

### Package A Helm Chart

- Use `gnupg` or `gpg` for encryption. Helm supports an older version of gnupg for encryption.

For development use - `gpg --quick-generate-key "John Smith" `
For production use - `gpg --full-generate-key "John smith"`

- Use command `helm package ` and `helm verify` to package a helm chart directory to tgz, and also
  verify that downloaded package shasum256 matches what is printed in the `verify` command.
- Typically, both the tgz file and the helm provenance file are uploaded together in the git repo.
- After packaging, generate the index file.

### What is the helm provenance file?

A helm provenance file (.prov) file that gets created along with package. This helps users verify
cryptographic signatures to avoid compromised files.

`helm verify` command validates that a publicly downloaded helm chart contains a valid provenance
file.

### Generate an index file

- Create a `<my-chart-name>-chart-files` directory in the repo, move the tgz and prov files here.
- Run the `helm repo index` command providing the url where chart will be uploaded to
  and path to chart files directory.

### Uploading a helm chart online

A helm chart repo typically contains the following:

- A package, a tgz archive file that was previously packaged.
- A generated index.yaml file.
- A helm provenance file (.prov) file.

* Upload the chart to anywhere you want - AWS S3 bucket, Azure blob etc., gh pages
* Once you upload, get the final link where chart was uploaded.

The users will now run the following commands to download and install the chart:

1. `helm repo add webapp-nginx-chart https://example.com/charts`
2. `helm repo list`
3. `helm install nginx-release webapp-nginx-chart/nginx-chart`

