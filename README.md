# lancache

![Version: 0.0.1](https://img.shields.io/badge/Version-0.0.1-informational?style=flat-square) ![Type: application](https://img.shields.io/badge/Type-application-informational?style=flat-square)

A Helm chart to deploy [lancachenet](https://github.com/lancachenet) DNS and monolithic cache services for caching game and content delivery downloads on Kubernetes.

## Overview

This chart provides a full deployment of `lancachenet` with:

* **lancache-dns**: Custom DNS server to redirect traffic to the cache
* **lancache-monolithic**: Single-container HTTP/S cache server
* **Persistent storage** for cached content and logs
* Exposed via a **LoadBalancer IP** for DNS resolution from clients

## Requirements

* Kubernetes: `>=1.16.0`
* Persistent volumes available for caching and logs
* A static IP assigned if using LoadBalancer (recommended)

## Configuration

See [values.yaml](./values.yaml) for all available options.

### DNS Setup

After installation, configure your network (router or PC) to use the LoadBalancer IP as its **primary DNS** server.

If `loadBalancerIP` is not set, retrieve the IP with:

```bash
kubectl get svc lancache -o jsonpath="{.status.loadBalancer.ingress[0].ip}"
```

## Persistence

This chart creates two `PersistentVolumeClaims`:

* **Cache**: stores downloaded content
* **Logs**: stores nginx logs

Ensure your cluster's storage class supports `ReadWriteOnce`.

## Values

| Key                                | Type   | Default                          | Description                                                    |
| ---------------------------------- | ------ | -------------------------------- | -------------------------------------------------------------- |
| envFile.USE\_GENERIC\_CACHE        | bool   | true                             | Set true for LoadBalancer / monolithic setup                   |
| envFile.LANCACHE\_IP               | string |                                  | IP(s) DNS will resolve for cache (space-separated if multiple) |
| envFile.DNS\_BIND\_IP              | string |                                  | IP DNS server binds to                                         |
| envFile.UPSTREAM\_DNS              | string | 8.8.8.8                          | Upstream resolver for unknown DNS queries                      |
| envFile.CACHE\_ROOT                | string | /data/cache                      | Path for storing cached data                                   |
| envFile.CACHE\_DISK\_SIZE          | string | 2000g                            | Max size of cache on disk                                      |
| envFile.MIN\_FREE\_DISK            | string | 10g                              | Minimum free disk space to maintain                            |
| envFile.CACHE\_INDEX\_SIZE         | string | 500m                             | Nginx index memory size (based on cache size)                  |
| envFile.CACHE\_MAX\_AGE            | string | 3650d                            | Max age to keep cached content                                 |
| envFile.TZ                         | string | Europe/Istanbul                  | Container timezone (e.g. Europe/Istanbul)                      |
| dns.image                          | string | lancachenet/lancache-dns\:latest | DNS server image                                               |
| dns.udpPort                        | int    | 53                               | UDP port to expose (DNS)                                       |
| dns.tcpPort                        | int    | 53                               | TCP port to expose (DNS)                                       |
| dns.service.type                   | string | LoadBalancer                     | Service type for external access                               |
| dns.service.loadBalancerIP         | string |                                  | Static LoadBalancer IP                                         |
| monolithic.image                   | string | lancachenet/monolithic\:latest   | Monolithic cache image                                         |
| monolithic.httpPort                | int    | 80                               | HTTP port for cache                                            |
| monolithic.httpsPort               | int    | 443                              | HTTPS port for cache                                           |
| monolithic.persistence.enabled     | bool   | true                             | Enable persistent volumes                                      |
| monolithic.persistence.cacheVolume | string | lancache-cache                   | Cache volume name                                              |
| monolithic.persistence.cacheSize   | string | 2000Gi                           | Size of cache volume                                           |
| monolithic.persistence.cachePath   | string | /data/cache                      | Mount path for cache                                           |
| monolithic.persistence.logsVolume  | string | lancache-logs                    | Logs volume name                                               |
| monolithic.persistence.logsSize    | string | 10Gi                             | Size of logs volume                                            |
| monolithic.persistence.logsPath    | string | /data/logs                       | Mount path for logs                                            |

## File Structure

| File                           | Description                                         |
| ------------------------------ | --------------------------------------------------- |
| `templates/deployment.yaml`    | Main deployment containing DNS and cache containers |
| `templates/service.yaml`       | Exposes DNS, HTTP, and HTTPS ports via Service      |
| `templates/env-configmap.yaml` | ConfigMap for environment variables                 |
| `templates/pvc.yaml`           | PersistentVolumeClaims for cache and logs           |

## Changelog

### \[0.0.1]

* Initial release with:

  * Monolithic + DNS in one deployment
  * Configurable LoadBalancer IP
  * Persistent cache and logs storage
  * Helm-native `values.yaml` support

