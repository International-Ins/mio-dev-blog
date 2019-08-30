---
layout: post
title: Why Kubernetes?
excerpt_separator: <!-- /excerpt -->
---

Kubernetes will help us drive down hosting costs, decrease maintenance time, and allow us to be cloud agnostic.
<!-- /excerpt -->
Sounds like a sales pitch, doesn't it?  Well, it's true.

Let's start with some definitions.

- [Kubernetes](https://kubernetes.io/) - Clustering software based on internal software at Google.  Developed by Google, with a plethora of [partners](https://kubernetes.io/partners/).
- Cloud Agnostic - Running in such a way as to not be dependent on a single cloud provider.  Kubernetes has managed implementations from [Digital Ocean](https://www.digitalocean.com/products/kubernetes/), [Amazon Web Services](https://aws.amazon.com/eks/), and [Microsoft Azure](https://docs.microsoft.com/en-us/azure/aks/), to name a few.
- Pod - A unit of work in Kubernetes.  This is usually a single Container, but sometimes may be multiple containers required for a task.

## Why Now?

Our hand was somewhat forced with Kubernetes.  Previously our systems had been using [Fleet](https://github.com/coreos/fleet) for our clustering tech, but it was deprecated several years ago.  With pending updates from AWS, and Fleet's deprecation status, we were forced to make a move.


## Why Kubernetes?

First off, Kubernetes provides an experience similar to managed solutions such as Amazon Elastic Beanstalk, or Heroku.  Application are build in a container (we use Docker containers), while configuration is stored in YAML files, and uploaded to Kubernetes for deployment.  Through the configuration, you tell Kubernetes what state your application should be in (resources, replicas, load balancers, storage, etc), and Kubernetes takes steps to make that state happen.

In addition, deployments then can then easily be managed by CI/CD software such as Jenkins, Travis, or [Gitlab](https://about.gitlab.com/) (our choice) using Kubernetes' baked in tools.  Once Gitlab and Kubernetes were wired together, deployments are as simple as pushing to `master` on Gitlab.

Kubernetes' ingress controllers (I chose [NGINX](https://www.nginx.com/)) also allow for many services to be hosted off a single load balancer.  In addition, the ingress controller is also attached to a cert-manager instance, which automagically obtains SSL certificates from [Let's Encrypt](https://letsencrypt.org/).  No more managing certs, or worrying about expiration.

## How is this better?

First and foremost, Kubernetes manages state, while Fleet only managed services.  This means that if a server is removed from the cluster, Kubernetes will automagically move pods from the removed server to a healthy server, keeping the application running.  If a server is added to the cluster, Kubernetes will begin using it.

In addition to managing the state of each application, Kubernetes also manages the state of the cluster itself.  To that end, services like [Datadog](https://www.datadoghq.com/) can be configured as cluster-level-state.  If you configure Datadog to manage logs for the cluster, then each server will automagically receive a copy of the pods required for Datadog.

Unlike similar offerings (Heroku, Elastic Beanstalk, Google App Engine), Kubernetes is cloud agnostic, so we could theoretically move our infrastructure from AWS to Azure, Digital Ocean, or even back to on-premises hosting, with the same settings we're currently using in AWS.

Another important feature:  Density.  With Fleet, each of our servers were dedicated to a single application.  Fleet allowed for co-locating some containers (cache server on the same machine as the application server), but for the most part the servers were single use only.  This meant having ~45 individual machines across our infrastructure.  With Kubernetes, I have been able to move all of these services to ~7 virtual machines, comprised of two Kubernetes clusters (one for Staging, one for Production), and database servers (RDS instances managed by AWS).  I am expecting ~60% savings in our cloud spending.

## Conclusion

Kubernetes is a wonderful solution for small teams managing their own DevOps, or larger teams where velocity and density are important.  With tools like [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/), you can get Kubernetes running on your machine today, and go play.  I highly suggest it.