# Cloud Native Buildpacks CI

This repository contains configuration for our CI deployment, at [ci.buildpacks.io](https://ci.buildpacks.io).

## Using assets generated by the pipeline

We generate [versioned lifecycle binaries for each commit](https://console.cloud.google.com/storage/browser/cloud-native-buildpacks-lifecycle-binaries/development)
(GCP account login required). You can also get a listing of binaries, without needing to login, by using [`gsutil`](https://cloud.google.com/storage/docs/gsutil):

```bash
$ gsutil ls -al gs://cloud-native-buildpacks-lifecycle-binaries/development/
``` 

We also provide a "latest" file to ease consumption by other automated systems, which can be directly accessed via HTTPS:

https://storage.googleapis.com/cloud-native-buildpacks-lifecycle-binaries/development/lifecycle-binaries-latest.tgz

https://storage.googleapis.com/cloud-native-buildpacks-lifecycle-binaries/release/lifecycle-binaries-latest.tgz

This file is updated after each successful acceptance test of the lifecycle.

To see and retrieve older versions of `lifecycle-binaries-latest.tgz`, you can use `gsutil`:

```bash
$ gsutil ls -al gs://cloud-native-buildpacks-lifecycle-binaries/development/lifecycle-binaries-latest.tgz
```

### Concourse

If you are using Concourse, here are the YAML snippets you need:

```yaml
resource_types:
- name: gcs-resource
  type: docker-image
  source:
    repository: frodenas/gcs-resource

resources:
- name: latest-lifecycle-archive-dev
  type: gcs-resource
  source:
    bucket: cloud-native-buildpacks-lifecycle-binaries
    # A GCP service account token with "Storage Legacy Bucket Viewer" and "Storage Object Viewer"
    json_key: ((concourse-service-account-key-json))
    versioned_file: development/lifecycle-binaries-latest.tgz

jobs:
- name: jobbityjob
  plan:
  - get: latest-lifecycle-archive-dev
``` 

### Contents

The `lifecycle-binaries-latest.tgz` archive contains:
* Lifecycle binaries without versions appended to their name, and
* A `VERSION` file.

For each file we zero out the `mtime`, `user` and `group` values to improve reproducibility.
