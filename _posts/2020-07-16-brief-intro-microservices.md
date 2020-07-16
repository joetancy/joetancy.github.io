---
title: "Brief introduction to microservices"
date: 2020-07-16T11:33:16
categories:
  - blog
---

Microservices architecture is really popular these days.
Running a collection of single-purpose programs benefits the whole application. This article gives you a brief overview of microservices architecture.

This makes the application:

- Loosely coupled
- Easily maintainable
- Independently deployable

They usually communicate with each other with APIs through HTTP.
Think of microservices as many workers with really specific job scope that comes together to form the application.

## Application

First, we'll have to write our application. There are a few frameworks that are suitable for writing microservices in many languages, so pick any you're comfortable with.

- Java
  - [Spring Boot](https://spring.io/projects/spring-boot)
  - [Spark](http://sparkjava.com/)
- Python
  - [Flask](https://flask.palletsprojects.com/en/1.1.x/)
  - [Bottle](https://bottlepy.org/docs/dev/)

The common attribute in frameworks is that they are slim and lightweight.

## Containers

Microservices usually runs in containers and a container orchestration platform.
One of the more popular containerisation and orchestration platforms would be [Docker](https://www.docker.com/), and [Kubernetes](https://kubernetes.io/).

Containers are self-contained images that include all dependencies that your service needs to run. These include your OS, web server, and your code that runs on the webserver. This image is built by Docker and uploaded to an image repository of your choice.
(See [Dockerhub](https://hub.docker.com/) or [AWS ECR](https://aws.amazon.com/ecr/))

Having multiple services means multiple container images to run. Kubernetes makes running these images easier by automating the deployment and scaling of these containers.

## Container Orchestration

By putting multiple pods together, it forms a cluster. Kubernetes runs clusters of pods. Pods consist of various containers.

Putting this in context, for example, you have an e-commerce application, designed in a microservices architecture.

- Cart
- Products
- User
- Tracking

Each of these is a container image running in the pods of a Kubernetes cluster.

The scalability of Kubernetes comes in when you specify e.g. "Cart" with very high traffic, to be able to scale to 5 more replicas. Kubernetes will measure the resource usage of "Cart", and if it hits a certain percentage, it spins up more instance of this "Cart" container in the same pod to share the load.

## Conclusion

With the application running in containers, which runs in the pods of Kubernetes, we now have a collection of services that runs specific jobs that can communicate with one another within the service cluster.

There will be a guide on how to containerise your service, deploy it, and run in Kubernetes.
