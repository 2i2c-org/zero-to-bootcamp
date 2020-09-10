# Technology overview

This section is to ensure that you have an overview of various tools and technology that you will encounter. With such overview in place, you are much better equipped to understand and architect a JupyterHub based infrastructure.



## Prerequisite tech

This tech is a prerequisite to learn and understand other tech.

### yaml
[YAML]() is like [JSON](): a language to format data in text. YAML is used with many tools (`kubectl`, `helm`, `hubploy`, `jupyter-book`) so it is well worth ten minutes to get introduced to its syntax.

### Git and GitHub
Basic knowledge of [git]() is assumed, such as knowing how to _clone_, _commit_, and _push_. To also understand the GitHub concept of a _Fork_ and _Pull Request_ is great.

### docker
Basic knowledge of [docker]() is assumed, such as knowing that a _Dockerfile_ can be built into an _image_ which when run is referred to as a _container_.



## Main tech

### JupyterHub
[JupyterHub]() is a web-server that allows its users to get access to a personal Jupyter server.
- JupyterHub _authenticates_ (who?) and _authorize_ (access?) its users with a [JupyterHub Authenticator]().
- JupyterHub _spawns_ Jupyter servers for its users with a [JupyterHub Spawner]().
- JupyterHub _routes_ network access to the user servers with a [JupyterHub Proxy]().

### Kubernetes
An administrator of a [Kubernetes]() cluster acts as an orchestrator of containers and their required networking and storage. In other words, the administrator can declare intentions and rely on the Kubernetes clusters machinery to fulfill the intentions. Intentions can for example be to always have two containers running, with 10GB persistent storage mounted, and to have a public IP address route incoming traffic to a running and ready container.

In practice administrators will register _Kubernetes resources_ representing their intentions to the _Kubernetes API-server_, and following that the _Kubernetes controllers_ for respective resource types work to ensure that this intent is met. As an example, an administrator can register a _Deployment resource_ with the API-server using `kubectl apply -f my-deployment-resource.yaml`. Following this, the _Deployment controller_ part of the Kubernetes machinery will notice that and take responsibility to meet the intent of the Deployment resource: to always have a set of containers running.

#### Essential Kubernetes terminology
- A _Pod resource_ is the intent to have a container running.
- A _Persistent Volume Claim (PVC) resource_ is the intent to allocate storage that can be used by a pod's container.
- A _Node resource_ represents registered servers where pods can be scheduled to run.

### Helm
`helm` is a tool to bundle Kubernetes _resource templates_ and _default values_ into a _Helm chart_. It makes reuse, adjustments, and installing or upgrading, a collection of Kubernetes resources a lot simpler.

The main job `helm` does is to _render_ a Helm chart's resource templates using a combination of values, both the default values part of the Helm chart and values passed by the user of the Helm chart. When Helm installs or upgrades a Helm chart, it renders the Helm chart's templates and communicates with the Kubernetes API-server to register or update the rendered Kubernetes resources.

#### Essential Helm terminology
- A _release_ represent an installation of a Helm chart into a Kubernetes cluster.
- A _revision_ represent a version of a release.

### The official JupyterHub Helm chart
The [jupyterhub/zero-to-jupyterhub-k8s]() project is both [a how-to guide]() as well as a Helm chart to make it easy to customize and install a JupyterHub into a Kubernetes cluster.



## Other tech

### hubploy
[hubploy]() can automate the process of:
- using `repo2docker` to build a Docker image
- using `sops` to decrypt secret configuration for a local Helm chart
- using `helm` to deploy a local Helm chart

### repo2docker
`repo2docker` is a tool to 

### sops

### jupyter-book
[]

### GitHub actions
By putting `.yaml` files in a GitHub repository's `.github/workflows` folder, instructions within the files will automatically be followed by GitHub's automated system referred to as _GitHub Actions_. A `jupyter-book` can for example be automatically built into HTML files and published by _GitHub Pages_ - GitHub's free service to publish basic websites.

### grafana


### prometheus


### Google cloud
  #### NFS / Filestore
  #### Persistent disks
  #### Cloud NAT
  #### Cloud KMS
