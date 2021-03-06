# Skuber

Skuber is a Scala client library for [Kubernetes](http://kubernetes.io). It provides a fully featured, high-level and strongly typed Scala API for managing Kubernetes cluster resources (such as Pods, Services, Deployments, ReplicaSets, Ingresses  etc.) via the Kubernetes REST API server.

## Features

- Comprehensive set of case classes for representing core and extended Kubernetes resource kinds
- Full support for converting resources between the case class and standard JSON representations 
- Client API for creating, reading, updating, removing, listing and watching resources on a Kubernetes cluster
- The API is asynchronous and strongly typed e.g. `k8s get[Deployment]("nginx")` returns a value of type `Future[Deployment]`
- Fluent API for creating and updating specifications of Kubernetes resources
- Uses standard `kubeconfig` files for configuration - see the [Configuration guide](docs/Configuration.md) for details

See the [programming guide](docs/GUIDE.md) for more details.

## Example Usage

This example creates a nginx service (accessed via port 30001 on each Kubernetes cluster node) that is backed by five nginx replicas.

    import skuber._
    import skuber.json.format._

    val nginxSelector  = Map("app" -> "nginx")
    val nginxContainer = Container("nginx",image="nginx").exposePort(80)
    val nginxController= ReplicationController("nginx",nginxContainer,nginxSelector)
    	.withReplicas(5)
    val nginxService = Service("nginx")
    	.withSelector(nginxSelector)
    	.exposeOnNodePort(30001 -> 80) 

    import scala.concurrent.ExecutionContext.Implicits.global

    val k8s = k8sInit

    val createOnK8s = for {
      svc <- k8s create nginxService
      rc  <- k8s create nginxController
    } yield (rc,svc)

    createOnK8s onComplete {
      case Success(_) => System.out.println("Successfully created nginx replication controller & service on Kubernetes cluster")
      case Failure(ex) => System.err.println("Encountered exception trying to create resources on Kubernetes cluster: " + ex)
    }

    k8s.close


## Prerequisites

The client supports v1.0 through v1.3 of the Kubernetes API, so should work against all supported Kubernetes releases in use today.

You need Java 8 to run Skuber.

## Release

You can include the current Skuber release in your application by adding the following to your `sbt` project:

    resolvers += Resolver.url(
      "bintray-skuber",
      url("http://dl.bintray.com/oriordan/skuber"))(
      Resolver.ivyStylePatterns)

    libraryDependencies += "io.doriordan" %% "skuber" % "1.3.0"

The current release has been built for Scala 2.11 - other Scala language versions such as 2.12 will be supported in future.

## Building

Building the library from source is very straightforward. Simply run `sbt test`in the root directory of the project to build the library (and examples) and run the unit tests to verify the build.

## Quick Start

The quickest way to get started with Skuber:

- If you don't already have Kubernetes installed, then follow the instructions [here](https://github.com/kubernetes/minikube) to install minikube, which is now the recommended way to run Kubernetes locally.

- Ensure Skuber configures itself from the default Kubeconfig file (`$HOME/.kube/config`) : 

	`export SKUBER_CONFIG=file` 

- Try one or more of the examples: if you have cloned this repository run `sbt` in the top-level directory to start sbt in interactive mode and then:

```
    > project examples

    > run
    [warn] Multiple main classes detected.  Run 'show discoveredMainClasses' to see the list

    Multiple main classes detected, select one to run:

    [1] skuber.examples.deployment.DeploymentExamples
    [2] skuber.examples.fluent.FluentExamples
    [3] skuber.examples.guestbook.Guestbook
    [4] skuber.examples.scale.ScaleExamples
    [5] skuber.examples.ingress.NginxIngress

    Enter number: 
```

For other Kubernetes setups, see the [Configuration guide](docs/Configuration.md) for details on how to tailor the configuration for your clusters security, namespace and connectivity requirements.

## Status

The coverage of the core Kubernetes API functionality by Skuber is extensive.

Support of more recent extensions group functionality is not yet entirely complete:  full support (with examples) is included for [Deployments](http://kubernetes.io/docs/user-guide/deployments/), [Horizontal pod autoscaling](http://kubernetes.io/docs/user-guide/horizontal-pod-autoscaling/) and [Ingress / HTTP load balancing](http://kubernetes.io/docs/user-guide/ingress/); support for other [Extensions API group](http://kubernetes.io/docs/api/#api-groups) features including [Daemon Sets](http://kubernetes.io/docs/admin/daemons/) and [Jobs](http://kubernetes.io/docs/user-guide/jobs/) is added over time.

## License

This code is licensed under the Apache V2.0 license, a copy of which is included [here](LICENSE.txt).
