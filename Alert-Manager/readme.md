# Alert Manager

The Alertmanager handles alerts sent by client applications such as the Prometheus server. It takes care of deduplicating, grouping, and routing them to the correct receiver integration such as email, PagerDuty, or OpsGenie. It also takes care of silencing and inhibition of alerts.

## Prerequistes

- A functioning kubernetes cluster
- A Prometheus server up and running
  \*If you don't have a Prometheus up you can do so by reviewing the [Prometheus] folder.

## Contents

- Config Map
- Deployment
- Service
- Template Config Map

## Install

Clone this repo to your host and then apply command :

```
kubectl apply -f Alert-Manager/
```

This will apply to all files in the folder you just cloned.

## Docs

You can find the docs at the official [Alert-Manager] page.

[prometheus]: https://github.com/manueldruart/kubernetes/tree/main/Prometheus
[alert-manager]: https://prometheus.io/docs/alerting/latest/overview/
