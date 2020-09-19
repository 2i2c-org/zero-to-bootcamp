# Technology overview

This section is to ensure that you have an overview of various tools and technology that you will encounter. With such overview in place, you are much better equipped to understand and architect a JupyterHub based infrastructure.



## Prerequisite tech

This tech is a prerequisite to learn and understand other tech.

````{margin}
```{admonition} Learn YAML
- [A 10 minute introduction.](https://www.youtube.com/watch?v=cdLNKUoMc6c).
```
````

### yaml

[YAML](https://yaml.org/) is like JSON: a language to format data in text. YAML is used with many tools (`kubectl`, `helm`, `hubploy`, `jupyter-book`) so it is well worth ten minutes to get introduced to its syntax.

````{margin}
```{admonition} Learn Git
- [A thorough introduction.](https://www.pluralsight.com/courses/how-git-works).
```
````

### Git and GitHub
Basic knowledge of [git](https://git-scm.com/) is assumed, such as knowing how to _clone_, _commit_, and _push_. To also understand the GitHub concept's of a _Fork_ and _Pull Request_ is great.

````{margin}
```{admonition} Learn Docker
- [A 11 minute introduction.](https://www.youtube.com/watch?v=gAkwW2tuIqE).
```
````

### docker
Basic knowledge of [docker](https://www.docker.com/) is assumed, such as knowing that a _Dockerfile_ can be built into an _image_ which when run is referred to as a _container_.


````{margin}
```{admonition} Learn Markdown
- [A quick Markdown tutorial.](https://commonmark.org/help/tutorial/).
```
````

### Markdown
_Markdown_ is a way to format text used with GitHub, Jupyter notebooks, and `jupyter-book`. To recognize Markdown formatted text and be able to write headers and code sections is generally useful.


## Main tech

### JupyterHub
[JupyterHub](https://jupyterhub.readthedocs.io/en/latest/) is a web-server that allows its users to get access to a personal Jupyter server.
- JupyterHub _authenticates_ (who?) and _authorize_ (access?) its users with a [JupyterHub Authenticator](https://jupyterhub.readthedocs.io/en/latest/reference/authenticators.html).
- JupyterHub _spawns_ Jupyter servers for its users with a [JupyterHub Spawner](https://jupyterhub.readthedocs.io/en/latest/reference/spawners.html).
- JupyterHub _routes_ network access to the user servers with a [JupyterHub Proxy](https://jupyterhub.readthedocs.io/en/latest/reference/proxy.html).

````{margin}
```{admonition} Learn Kubernetes
- [A good initial introduction.](https://www.youtube.com/watch?v=4ht22ReBjno).
- [A good followup introduction.](https://www.youtube.com/watch?v=QJ4fODH6DXI)
- [A thorough continuation.](https://vimeo.com/245778144/4d1d597c5e)
```
````

### Kubernetes
An administrator of a [Kubernetes](https://kubernetes.io/) cluster acts as an _orchestrator_ of containers and their required networking and storage. In other words, the administrator can declare intentions and rely on the Kubernetes clusters machinery to fulfill the intentions. Intentions can for example be to always have two containers running, with 10GB persistent storage mounted, and to have a public IP address route incoming traffic to a running and ready container.

In practice administrators will register _Kubernetes resources_ representing their intentions to the _Kubernetes API-server_, and following that the _Kubernetes controllers_ for the respective resource types will work to ensure that this intent is met. As an example, an administrator can register a _Deployment resource_ with the API-server using `kubectl apply -f my-deployment-resource.yaml`. Following this, the _Deployment controller_ that is part of the Kubernetes machinery will notice that and take responsibility to meet the intent of the Deployment resource: to always have a set of containers running.

#### Essential Kubernetes terminology
- A _Pod resource_ is the intent to have a container running.
- A _Persistent Volume Claim (PVC) resource_ is the intent to allocate storage that can be used by a pod's container.
- A _Node resource_ represents registered servers where pods can be scheduled to run.

````{margin}
```{admonition} Learn Helm
- [A 14 minute introduction.](https://www.youtube.com/watch?v=-ykwb1d0DXU)
```
````

### Helm
[Helm](https://helm.sh/) makes reuse, adjustments, and installing or upgrading, a collection of Kubernetes resources a lot simpler. `helm` is a command line tool to bundle Kubernetes _resource templates_ and _default values_ into a _Helm chart_.

Helm's main job is to _render_ a Helm chart's resource templates, this is done using values explicitly passed in addition to the default values part of the Helm chart. When Helm installs or upgrades a Helm chart, it first renders the templates and then communicates with the Kubernetes API-server to register or update the rendered Kubernetes resources. Another feature of Helm is to enable _rollbacks_: to be able to revert to a previous state of an installed Helm chart.

#### Essential Helm terminology
- A _release_ represent an installation of a Helm chart into a Kubernetes cluster.
- A _revision_ represent a version of a release.

### The JupyterHub Helm chart
The JupyterHub Helm chart makes it easier to install JupyterHub in a Kubernetes cluster. It is [developed on GitHub](https://github.com/jupyterhub/zero-to-jupyterhub-k8s), [published online](), and comes with a [how-to guide](https://z2jh.jupyter.org).

The JupyterHub Helm chart installs more than just a container running JupyterHub in the Kubernetes cluster, so here is an overview of the Kubernetes Pods you will find.

- `hub` - A JupyterHub server.
  - [KubeSpawner](https://github.com/jupyterhub/kubespawner) is the _JupyterHub Spawner_, spawning user Pods.
  - [ConfigurableHTTPProxy (CHP, Python class)](https://github.com/jupyterhub/jupyterhub/blob/1.1.0/jupyterhub/proxy.py#L411) is the _JupyterHub Proxy representation_ used by JupyterHub, which speak with the actual CHP server in another pod.
- `proxy` - A [ConfigurableHttpProxy (CHP, NodeJS server)](https://github.com/jupyterhub/configurable-http-proxy) server that routes traffic as dynamically configured by the JupyterHub server's Proxy representation.
- `autohttps` - A network proxy for incoming traffic that enables HTTPS communication. It can both acquire and updates TLS certificates from [Let's Encrypt](https://letsencrypt.org/) and manage decryption/encryption of incoming/outgoing traffic.
- `user-scheduler` - A configured [kube-scheduler](https://kubernetes.io/docs/concepts/scheduling-eviction/kube-scheduler/) server that ensures that the spawned user Pods are packed tight on the nodes, which especially improves the ability to scale down nodes as new users don't keep spreading out.
- `user-placeholder` - A set of dummy Pods to warm seats for user Pods. They trigger automatic autoscaling that adds nodes to the cluster if needed and if possible, and they get _evicted_ in favor of actual user Pods when needed for a user Pod to fit.
- `continuous-image-puller` - A dummy Pod downloading the image(s) used by the user Pods ahead of time on each node where users can schedule.
- `hook-image-puller` - A temporary Pod like the `continuous-image-puller`, but to ensure images are pulled to nodes before a JupyterHub Helm chart reconfigures JupyterHub to start user pods with a new version of an image.



## Other tech

### hubploy
[hubploy](https://github.com/yuvipanda/hubploy) can automate the process of:
- using `repo2docker` to build a Docker image
- using `sops` to temporary decrypt secret configuration for a local Helm chart
- using `helm` to deploy a local Helm chart

### repo2docker
[`repo2docker`](https://github.com/jupyterhub/repo2docker) is a tool to build a Docker image based on configuration files like `requirements.txt`. It is what's used in BinderHub's like [mybinder.org](https://mybinder.org) for example.

### sops
[`sops`](https://github.com/mozilla/sops) is a tool integrating with some _key management system_ (KMS), and this can help you avoid needing to have encrypted content in a git repository. To store encrypted content in a git repository has downsides related to it being hard to manage situations when a decryption key is exposed.

### jupyter-book
[`jupyter-book`](https://github.com/executablebooks/jupyter-book) is a command line tool with [excellent documentation](https://jupyterbook.org) that help you all the way to get a good looking published website based on Jupyter notebooks and Markdown files.

### GitHub Actions
By putting `.yaml` files in a GitHub repository's `.github/workflows` folder, instructions within the files will automatically be followed by GitHub's automated system referred to as _GitHub Actions_. A `jupyter-book` can for example be automatically built into HTML files and published by _GitHub Pages_ - GitHub's free service to publish basic websites.

````{margin}
```{admonition} Learn Grafana and Prometheus
- [Live demo of Grafana.](https://play.grafana.org/)
- [A 21 minute introduction to Prometheus.](https://www.youtube.com/watch?v=h4Sl21AKiDg)
```
````

### Grafana and Prometheus
To use [Prometheus]() and [Grafana]() is not required alongside alongside a JupyterHub deployment, but they can provide an administrator with relevant insights, such as how many users are active and if things are running as they should.

#### Collecting metrics - Prometheus
Prometheus is used to collect _metrics_ into a series of values that change over time, which it does by asking JupyterHub and other servers repeatedly. A metric from JupyterHub is for example how many running servers it currently manages. Prometheus is also responsible for storing this data for a certain time and exposing a way for Grafana and other services to ask it for information.

```{admonition} Good to know
- JupyterHub expose its current metrics as plain text webpage under the `/hub/metrics` path. The plain text format follows a syntax specific to Promethus.
- [PromQL]() is a language to ask Prometheus for its collected metrics, which is used by Grafana.
```

#### Viewing metrics - Grafana
Grafana allows an administrator of a JupyterHub to view already collected _metrics_ in _dashboards_.

You can get by without needing to learn how to define your own dashboards by importing existing ones, such as [NeuroHackademy 2020's dashboards](https://github.com/neurohackademy/nh2020-jupyterhub/tree/master/grafana-dashboard-definitions)).

### Network File System (NFS)
For a JupyterHub deployment, a Network File System (NFS) based storage can be a useful for various reasons. Below are common reasons to use NFS alongside a JupyterHub deployment on Kubernetes.

1. __read/write storage for multiple user.__

   By using NFS, you can configure access to storage with both read and write access from multiple locations.
1. __To reduce overprovisioning storage.__

   By using NFS, you can let users share the provided space more efficiently. Instead of paying for a fixed amount of GB for each user, you can pay for the total storage needed by all users.

To use NFS storage, you need a NFS server and storage that this NFS server controls. Cloud providers provide managed NFS servers, such as [Google Filestore](), but you can also install your own NFS server, such as [NFS Ganesha](https://github.com/nfs-ganesha/nfs-ganesha/wiki) suggestably using [this project](https://github.com/kubernetes-sigs/nfs-ganesha-server-and-external-provisioner/pull/14/files) in a Kubernetes cluster.

### Network Address Translation (NAT)
Network Address Translation (NAT) allows mapping of IP addresses and ports to other IP addresses and ports. This can be needed or useful for various reasons.

1. __A single public IP for multiple devices.__

   Both a public IP address and a device performing NAT is needed to allow devices with [private IP addresses](https://en.wikipedia.org/wiki/Private_network) in a Local Area Network (LAN) to communicate with devices on the internet. This is because a network packet on the internet must have a declared origin that other devices can reach to respond, and a private IP address isn't going to be unique as a public IP address provided by an internet provider.

1. __A predefined set of public IPs.__

   If you go through a NAT to reach internet rather than an ephemeral machine, you will likely be able to control what public IPs the NAT will use. This can be useful if you want to configure a database to accept connections from a specific set of IPs for example. It can also be useful to ensure you have a collection of public IPs when accessing the internet, so services like GitHub receiving traffic doesn't think that only one IP is sending all traffic, because then they may think their service is being abused.

### Key Management System (KMS)

TODO
