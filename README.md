# Setup
## Angular8 project CI / CD with Github and Google Cloud
#### Source : [Continuous Delivery in Google Cloud Platform — Cloud Build with App Engine](https://medium.com/google-cloud/continuous-delivery-in-google-cloud-platform-cloud-build-with-app-engine-8355d3a11ff5)

### 1. New angular project
```bash
ng new angular8-ci-cd-example
```

### 2. Configure "Cloud Build service" at the root of the angular project
> `cloudbuild.yaml`
```yaml

steps:
- name: 'gcr.io/cloud-builders/npm:node-10.10.0'
  id: 'Install'
  args: [ install ]

- name: 'gcr.io/cloud-builders/npm:node-10.10.0'
  id: 'Prod build'
  args: [ run, build, --prod ]

- name: 'gcr.io/cloud-builders/npm:node-10.10.0'
  id: 'Test'
  args: [ run, 'test' ]

- name: 'gcr.io/cloud-builders/gcloud'
  args: [ app, deploy ]
```

### 2. Configure the deployment at the root of the angular project
> `app.yaml`
```yaml

runtime: python27
api_version: 1
threadsafe: yes

handlers:

# Resources
- url: /(.*\.html)
  static_files: dist/angular8-ci-cd-example/\1
  upload: dist/angular8-ci-cd-example/(.*\.html)

# Resources
- url: /(.*\.(gif|png|jpg|css|js)(|\.map))$
  static_files: dist/angular8-ci-cd-example/\1
  upload: dist/angular8-ci-cd-example/(.*)(|\.map)

# Fonts
- url: /(.*\.(ttf|woff2|woff)(|\.map))$
  mime_type: font/opentype
  static_files: dist/angular8-ci-cd-example/\1
  upload: dist/angular8-ci-cd-example/(.*)(|\.map)

# Site root.
- url: /
  static_files: dist/angular8-ci-cd-example/index.html
  upload: dist/angular8-ci-cd-example/index.html

# Catch-all rule, responsible from handling Angular application routes (deeplinks).
- url: /.*
  static_files: dist/angular8-ci-cd-example/index.html
  upload: dist/angular8-ci-cd-example/index.html

skip_files:
- ^(?!dist)
```

### 3. Give `"App Engine Admin"` role to the project service account

### 4. Enable `"App Engine Admin API"` for the GCP Project

### 5. Create `"App Project"` with the desired region to host the application
```bash
gcloud app create
```
--------------------------------------------------------------------------------------
> ### Optional - At this point, you can deploy from the development machine straight to the production build. But if you're using a source manager like github, you may want to keep going and set this up too to automatically build the latest master / specific branches in production.
```bash
gcloud builds submit --config cloudbuild.yaml
```
--------------------------------------------------------------------------------------

### 6. Connect the GCP to an existing / new git repository
#### (Optional) Mirror existing repository to keep a source control on gcloud
> 1. Enable "`Cloud Source Repositories API`" for the GCP Project
> 2. `https://console.cloud.google.com/code/develop/repo?project=<your-project-id>`

#### Add trigger to either the mirrored repository or the original github one
> `https://console.cloud.google.com/cloud-build/triggers?project=<your-project-id>`

(For example, a trigger `ON PUSH` on the `MASTER` branch will trigger )
