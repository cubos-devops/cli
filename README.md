# Cubos CLI

This repository contains Docker images with a set of tools preinstalled and useful scripts to help simplify CI/CD pipelines, plus a GitHub Action to deploy to Kubernetes.

## Images

Images are published to the GitHub Container Registry (GHCR) by default.

| Image                                | Purpose                                                              | Contents                                                    |
| ------------------------------------ | -------------------------------------------------------------------- | ----------------------------------------------------------- |
| `ghcr.io/cubos-devops/cli`           | Full image with every tool and script (same as `docker.io/cubos/cli`) | [Preinstalled tools](#preinstalled-tools-full-image) and [all scripts](#additional-scripts) |
| `ghcr.io/cubos-devops/cli/k8s`       | Slim image to deploy to a Kubernetes cluster                         | `kubectl`, Node.js, `kube-templates`, `cluster_from_env` and `deploy` |
| `docker.io/cubos/cli`                | Docker Hub mirror of the full image, kept for backward compatibility | Same as `ghcr.io/cubos-devops/cli`                          |

Every image is tagged with `latest`, `<major>`, `<major>.<minor>` and `<version>` on each release.

### Preinstalled tools (full image)

- [AWS CLI](https://aws.amazon.com/cli/)
- [Bash](https://www.gnu.org/software/bash/)
- [Composer](https://getcomposer.org/)
- [Docker](https://www.docker.com/)
- [Firebase Tools](https://firebase.google.com/docs/cli)
- [Git](https://git-scm.com/)
- [Google Cloud SDK (gcloud CLI)](https://cloud.google.com/sdk/gcloud/)
- [Helm](https://helm.sh/)
- [jq](https://stedolan.github.io/jq/)
- [kubectl](https://kubernetes.io/docs/reference/kubectl/)
- [Node.js](https://nodejs.org/)
- [npm](https://www.npmjs.com/)
- [PHP](https://php.net/)
- [Python](https://www.python.org/)
- [s3cmd](https://s3tools.org/s3cmd)
- [yq](https://kislyuk.github.io/yq/)

### Slim image: `k8s`

Contains only what `cluster_from_env` and `deploy` need: Bash, curl, jq, yq, envsubst, kubectl, Node.js, `ts-node`, `ejs-cli` and `@cubos/kube-templates`. Use it instead of the full image when the job only deploys to Kubernetes.

## Additional scripts

Available in the full image. The `k8s` image only has `cluster_from_env` and `deploy`.

- `cluster_from_env`: Configure `kubectl` to use a Kubernetes cluster from environment variables.
- `deploy_app_engine`: Deploy an application to Google App Engine.
- `deploy_firebase_hosting`: Deploy a static website to Firebase Hosting.
- `deploy_site`: Deploy a static website to either Firebase Hosting or Google Cloud Storage.
- `deploy`: Deploy YAML and _kube-templates_ files to a Kubernetes cluster.
- `docker_build`: Build a Docker image and push it to a registry, using buildx, with support for multiple architectures.
- `docker_login`: Login to a Docker registry.
- `gsutil_auth_from_env`: Authenticate to Google Cloud using a service account JSON key encoded in Base64 stored in an environment variable.
- `s3cfg_from_env`: Generate a `.s3cfg` file from environment variables.
- `undeploy_site`: Delete a static website from Google Cloud Storage.
- `upload_chart`: Upload a Helm chart to Nexus Repository Manager.

## Example usage

### GitLab CI

```yaml
Deploy:
  stage: deploy
  image: ghcr.io/cubos-devops/cli/k8s
  only:
    - main
  script:
    - cluster_from_env CLUSTER_PRD
    - deploy
```

Use `image: ghcr.io/cubos-devops/cli` (or `cubos/cli`) when the job needs the other tools.

### GitHub Actions

Using the action, which runs `cluster_from_env` and then `deploy` in a single step:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: cubos-devops/cli@v1
        with:
          cluster: CLUSTER_PRD
        env:
          CLUSTER_PRD_K8S_SERVER_URL: ${{ secrets.CLUSTER_PRD_K8S_SERVER_URL }}
          CLUSTER_PRD_K8S_SERVER_CA: ${{ secrets.CLUSTER_PRD_K8S_SERVER_CA }}
          CLUSTER_PRD_K8S_TOKEN: ${{ secrets.CLUSTER_PRD_K8S_TOKEN }}
```

The credentials can also be passed as inputs instead of environment variables:

```yaml
      - uses: cubos-devops/cli@v1
        with:
          server-url: ${{ secrets.K8S_SERVER_URL }}
          server-ca: ${{ secrets.K8S_SERVER_CA }}
          token: ${{ secrets.K8S_TOKEN }}
```

| Input               | Default   | Description                                                                          |
| ------------------- | --------- | ------------------------------------------------------------------------------------ |
| `cluster`           | `CLUSTER` | Prefix of the `<cluster>_K8S_*` environment variables (argument of `cluster_from_env`) |
| `server-url`        |           | API server URL (alternative to `<cluster>_K8S_SERVER_URL`)                           |
| `server-ca`         |           | Base64 CA data (alternative to `<cluster>_K8S_SERVER_CA`)                            |
| `token`             |           | Service account token (alternative to `<cluster>_K8S_TOKEN`)                         |
| `server-name`       |           | Cluster name in the kubeconfig (alternative to `<cluster>_K8S_SERVER_NAME`)          |
| `working-directory` | `.`       | Directory that contains the `deploy/` folder                                         |
| `deploy`            | `true`    | Set to `false` to only configure `kubectl`                                           |

The kubeconfig lives inside the action's container, so `kubectl` is not configured for later steps. To run your own commands against the cluster, use the image directly as a job container:

```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    container: ghcr.io/cubos-devops/cli/k8s
    steps:
      - uses: actions/checkout@v4
      - run: |
          cluster_from_env CLUSTER_PRD
          kubectl get pods -A
          deploy
        env:
          CLUSTER_PRD_K8S_SERVER_URL: ${{ secrets.CLUSTER_PRD_K8S_SERVER_URL }}
          CLUSTER_PRD_K8S_SERVER_CA: ${{ secrets.CLUSTER_PRD_K8S_SERVER_CA }}
          CLUSTER_PRD_K8S_TOKEN: ${{ secrets.CLUSTER_PRD_K8S_TOKEN }}
```

## Releasing

Publishing a GitHub release builds and pushes every image, and moves the `vMAJOR` tag (e.g. `v1`) to the new release so `cubos-devops/cli@v1` follows it. The action pulls `ghcr.io/cubos-devops/cli/k8s:<major>`, so keep the action's major version and the image's in step.

Because `action.yml` is at the repository root, GitHub detects the action and offers **Publish this Action to the GitHub Marketplace** when drafting a release.

## Contributing

PRs are welcome! Feel free to improve current scripts or add new ones.

By submitting a PR, you agree to license your work under the [GNU Lesser General Public License v3.0](LICENSE).
