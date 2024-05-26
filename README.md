This project is based on: https://github.com/osodevops/kafka-gitops-examples 

# Kafka GitOps Example

A Kafka / Confluent GitOps workflow example for multi-env deployments with Flux, Kustomize, Helm and Confluent Operator

---

## Usage

For this example we assume a single cluster simulating a production confluent environment. The end goal is to leverage Flux and Kustomize to manage [Confluent Operator for Kubernetes](https://github.com/confluentinc/operator-earlyaccess). You can extend with another cluster while minimizing duplicated declarations.

We will configure [Flux](https://fluxcd.io/) to install, deploy and config the [Confluent Platform](https://www.confluent.io/product/confluent-platform) using their `HelmRepository` and `HelmRelease` custom resources.
Flux will monitor the Helm repository, and can be configured to automatically upgrade the Helm releases to their latest chart version based on semver ranges.

You may find this project helpful by simply referencing the documentation, code, and strategies for managing Kafka resources on Kubernetes. Additionally, if you just wish to operate a working example of the new Confluent operator, the following usage instructions will guide you.


### Repository structure

The Git repository contains the following top directories:

- **flux-system** dir contains the required kubernetes resources for flux to operate
- **kustomize/base** dir contains the base definition of the confluent stack.
- **kustomize/environments** dir containing an example environment, folders could be copied to create additional environments.  Files within are 'patches' which are layered on top of the definitions found in kustomize/base
- **kustomize/operator** dir the helm chart definition for confluent-for-kubernetes (CFK).


```
├── flux-system
├── kustomize
│   ├── base
│   │   ├── confluent
│   ├── environments
│   │   └── sandbox
│   └── operator
```

### Forking this repository.
In order to showcase the GitOps behaviour of the FluxCD toolkit you will require the ability to write to a repository.  Fork this repository, and update line 11 of the file `./flux-system/gotk-sync.yaml` to the new https git address of your forked repository.  Also make note of line 10 'branch'; this is the branch of the repository which Flux will monitor

### Deploy base Flux components
#### Overview
This step will install the base Flux kubernetes components onto your kubernetes cluster.  To inspect what is being applied, simply look through the contents of `./flux-system/gotk-components.yaml`.  You will see a mix of Custom Resource Definitions, Service Accounts, Deployments, and other various components.  After the application of these resource definitions is completed, you should see the following pods running:

* Helm-Controller
* Kustomize Controller
* Notification Controller
* Source Controller

For more information on what these controllers do, please review [the documentation here](https://fluxcd.io/docs/components/).




## Examples

### Deployment Process
* Navigate to `./flux-system`
* Run `kubectl apply -f gotk-components.yaml`


### Deploy Flux Sync
#### Overview
This next step will tell Flux what repository to monitor, and, within that repository, what kustomization files to start with.  The first Kustomize resource that Flux will look for to is located at `./kustomize/operator`.  This will install the confluent-for-kubernetes Helm chart.   After a successful health check of the operator (which will run as a pod), Flux will then proceed to deploy our first environment located at  `./kustomize/environments/sandbox`.

### Deployment Process
* Navigate to `./flux-system`
* run `kubectl apply -f gotk-sync.yaml`

#### Watch Flux in action!
Now that we have flux monitoring the forked Git repository, let's demonstrate the GitOps behaviour!  If everything has deployed successfully, you should see a healthy confluent stack looking like this:
```console
│ NAME                                          PF   READY      RESTARTS STATUS      IP              NODE         AGE        │
│ confluent-operator-global-7ffc5b469d-knmfj    ●    1/1               0 Running     172.17.0.7      minikube     21m        │
│ connect-0                                     ●    1/1               0 Running     172.17.0.17     minikube     9m31s      │
│ controlcenter-0                               ●    1/1               1 Running     172.17.0.11     minikube     21m        │
│ kafka-0                                       ●    1/1               3 Running     172.17.0.8      minikube     21m        │
│ kafka-1                                       ●    1/1               3 Running     172.17.0.10     minikube     21m        │
│ kafka-2                                       ●    1/1               3 Running     172.17.0.9      minikube     21m        │
│ ksqldb-0                                      ●    1/1               1 Running     172.17.0.12     minikube     21m        │
│ schemaregistry-0                              ●    1/1               1 Running     172.17.0.14     minikube     21m        │
│ zookeeper-0                                   ●    1/1               0 Running     172.17.0.15     minikube     21m        │
│ zookeeper-1                                   ●    1/1               0 Running     172.17.0.16     minikube     21m        │
│ zookeeper-2                                   ●    1/1               0 Running     172.17.0.13     minikube     21m        │
│
```
To exhibit Flux, let's change our kafka replicas from the default of 3, to 4:
* In the file `./kustomize/environments/sandbox/kafka.yaml` uncomment the line `#  replicas: 4`, commit that change to your repository (git), and push upstream.   The next time flux performs a 'sync' (observable in the 'source controller' logs), it will the change to the kafka spec, and in turn increase our kafka cluster from size '3' to '4'.

### Develop Locally
If you want to test configuration out locally without the need to push up to git (i.e. testing locally with Minikube), the deployment can be replicated very simply:

* Navigate to `./flux-system`
* Run `kubectl apply -f gotk-components.yaml`

**instead of deploying the gotk-sync.yaml, we'll perform the kubectl kustomize applies ourselves.**

* Navigate to `./kustomize/operator`
* Run `kubectl apply -k .`

**monitor the running pods, wait until the 'confluent-operator' pod is in a running state**

* Navigate to `./kustomize/environments/`
* Run `kubectl apply -k .`





## Related Projects

Check out these related projects.

- [Confluent for Kubernetes (CFK) examples](https://github.com/osodevops/confluent-kubernetes-playground) - Playground for Kafka / Confluent Kubernetes experimentations



## Need some help

File a GitHub [issue](https://github.com/osodevops/kafka-gitops-examples/issues), send us an [email][email] or [tweet us][twitter].

## The legals

Copyright © 2017-2022 [OSO](https://oso.sh) | See [LICENCE](LICENSE) for full details.

[<img src="https://oso-public-resources.s3.eu-west-1.amazonaws.com/oso-logo-green.png" alt="OSO who we are" width="250"/>](https://oso.sh/who-we-are/)

## Who we are

We at [OSO][website] help teams to adopt emerging technologies and solutions to boost their competitiveness, operational excellence and introduce meaningful innovations that drive real business growth. Our developer-first culture, combined with our cross-industry experience and battle-tested delivery methods allow us to implement the most impactful solutions for your business.

Looking for support applying emerging technologies in your business? We’d love to hear from you, get in touch by [email][email]

Start adopting new technologies by checking out [our other projects][github], [follow us on twitter][twitter], [join our team of leaders and challengers][careers], or [contact us][contact] to find the right technology to support your business.[![Beacon][beacon]][website]

  [logo]: https://oso-public-resources.s3.eu-west-1.amazonaws.com/oso-logo-green.png
  [website]: https://oso.sh?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=website
  [github]: https://github.com/osodevops?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=github
  [careers]: https://oso.sh/careers/?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=careers
  [contact]: https://oso.sh/contact/?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=contact
  [linkedin]: https://www.linkedin.com/company/oso-devops?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=linkedin
  [twitter]: https://twitter.com/osodevops?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=twitter
  [email]: mailto:enquiries@oso.sh?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=email
  [readme_header_img]: https://oso-public-resources.s3.eu-west-1.amazonaws.com/oso-animation.gif
  [readme_header_link]: https://oso.sh/what-we-do/?utm_source=github&utm_medium=readme&utm_campaign=osodevops/kafka-gitops-examples&utm_content=readme_header_link
  [beacon]: https://github-analyics.ew.r.appspot.com/G-WV0Q3HYW08/osodevops/kafka-gitops-examples?pixel&cs=github&cm=readme&an=kafka-gitops-examples
