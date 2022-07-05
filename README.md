### Deployments

Deployments refer to the instances of various  services running to use the single-cell pipeline, e.g. `ui`, `api`, `worker`
copies. Deployments are managed *automatically* by [Flux](http://fluxcd.io/).

### Flux

Flux is a GitOps Continuous Delivery platform. It is deployed on the cluster alongside all other base infrastructure. It
manages custom resources called *HelmRelease*. A HelmRelease contains data identifying the Helm Chart that is to be deployed,
as well as any custom configuration to the chart that may be necessary. Broadly, it is designed to automate manual
`helm install` or `helm upgrade` operations.

Flux is granted read and write access to this repository by the deploy script. This is by automatically creating/updating a
[deploy key](https://github.com/hms-dbmi-cellenics/iac/settings/keys) for this repository by the base infrastructure deployment
workflow.

### Releases

Flux running on the `$ENVIRONMENT` cluster is continuously scanning the `$ENVIRONMENT/` folder in this repo
for updates. When a change to a file in the `$ENVIRONMENT/` folder is found, Flux will automatically apply those
changes to the Helm chart release on the cluster. For example, if a `ui` deployment needs to run on a non-standard port,
the *HelmRelease* resource corresponding to this deployment is pushed to GitHub, and these changes are automatically picked up
applied on the cluster.

Each repository corresponding to a deployment is responsible for managing its own HelmRelease
resource in this repository. See the section on CI for more details, and [this](https://github.com/fluxcd/helm-operator-get-started)
explanation of how Flux works.

**Important**: Accordingly, this repo is automatically populated by the CI/CD
pipelines of various Biomage deployments. Do not commit, push, or submit a pull
request to change manifests in this repo. Pull requests attempting to modify
files except documentation will be automatically rejected.

### Continuous Integration

CI is managed by GitHub actions. Each repository has its own custom CI pipeline, but
they broadly follow a `Test > Build > Push image > Push HelmRelease`
pattern.

### Images

Docker images are built by CI and pushed to AWS ECR. Each repository should have a
container repository with the same name in ECR. This repository should be checked
and created by CI if it is not present.

### Deployment

Each repository is responsible for managing its own HelmRelease files. Each repository
that deploys a HelmRelease should have a `.flux.ci` file that contains the base template
that the CI pipeline will fill in and extend. The CI pipeline for a given project will
push this filled template to this repository for Flux to pick it up.

### CI privileges

Each CI pipeline needs a certain set of permissions from both GitHub and AWS to operate.
AWS privileges that should be available to the CI pipeline should be placed under `.ci.yaml`.

Changes to CI privileges must be deployed *manually* by someone with sufficient access
by running `biomage rotate-ci` using [biomage-utils](https://github.com/hms-dbmi-cellenics/biomage-utils).
