# Prometheus

<img src="https://upload.wikimedia.org/wikipedia/commons/thumb/3/38/Prometheus_software_logo.svg/2066px-Prometheus_software_logo.svg.png" width="100">

Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company. To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes.

Prometheus collects and stores its metrics as time series data, i.e. metrics information is stored with the timestamp at which it was recorded, alongside optional key-value pairs called labels.

## Prerequistes

- A functioning kubernetes cluster

## Contents

- Cluster Role
- Deployment
- Service
- Config Map

## Install

Clone this repo to your host and then apply command :

```
kubectl apply -f Prometheus/
```

This will apply to all files in the folder you just cloned.

## Docs

You can find the docs at the official [Prometheus] site.

[prometheus]: https://prometheus.io/docs/introduction/overview/
