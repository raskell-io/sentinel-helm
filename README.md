# Sentinel Helm Chart

Helm chart for deploying [Sentinel](https://github.com/raskell-io/sentinel) - a high-performance, security-focused reverse proxy built on Cloudflare's Pingora.

## Quick Start

```bash
# Add the repository (when published)
helm repo add sentinel https://charts.sentinel.raskell.io
helm repo update

# Install with default values
helm install sentinel sentinel/sentinel

# Install with custom configuration
helm install sentinel sentinel/sentinel -f values.yaml
```

## Installation from Source

```bash
git clone https://github.com/raskell-io/sentinel-helm.git
cd sentinel-helm
helm install sentinel .
```

## Configuration

See [values.yaml](values.yaml) for the full list of configuration options.

### Basic Example

```yaml
replicaCount: 2

config:
  raw: |
    listeners {
        listener "http" {
            address "0.0.0.0:80"
            protocol "http"
        }
    }

    routes {
        route "api" {
            matches { path-prefix "/api" }
            upstream "backend"
        }
    }

    upstreams {
        upstream "backend" {
            target "my-service:8080"
            health-check { path "/health" }
        }
    }
```

### Using an Existing ConfigMap

```yaml
config:
  existingConfigMap: my-sentinel-config
  configKey: sentinel.kdl
```

### Enabling Autoscaling

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### Prometheus Monitoring

```yaml
serviceMonitor:
  enabled: true
  interval: 30s
```

### Ingress

```yaml
ingress:
  enabled: true
  className: nginx
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
  hosts:
    - host: proxy.example.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: proxy-tls
      hosts:
        - proxy.example.com
```

## Parameters

### Global

| Parameter | Description | Default |
|-----------|-------------|---------|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Image repository | `ghcr.io/raskell-io/sentinel` |
| `image.tag` | Image tag | `""` (uses appVersion) |
| `image.pullPolicy` | Image pull policy | `IfNotPresent` |

### Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `config.raw` | Raw KDL configuration | `""` |
| `config.existingConfigMap` | Use existing ConfigMap | `""` |
| `config.configKey` | Key in ConfigMap | `sentinel.kdl` |

### Service

| Parameter | Description | Default |
|-----------|-------------|---------|
| `service.type` | Service type | `ClusterIP` |
| `service.httpPort` | HTTP port | `80` |
| `service.httpsPort` | HTTPS port | `443` |
| `service.metricsPort` | Metrics port | `9090` |

### Resources

| Parameter | Description | Default |
|-----------|-------------|---------|
| `resources.limits.cpu` | CPU limit | `1000m` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.requests.cpu` | CPU request | `100m` |
| `resources.requests.memory` | Memory request | `128Mi` |

### Security

| Parameter | Description | Default |
|-----------|-------------|---------|
| `securityContext.runAsNonRoot` | Run as non-root | `true` |
| `securityContext.runAsUser` | User ID | `65534` |
| `securityContext.readOnlyRootFilesystem` | Read-only root | `true` |

### Optional Features

| Parameter | Description | Default |
|-----------|-------------|---------|
| `autoscaling.enabled` | Enable HPA | `false` |
| `podDisruptionBudget.enabled` | Enable PDB | `false` |
| `serviceMonitor.enabled` | Enable Prometheus ServiceMonitor | `false` |
| `ingress.enabled` | Enable Ingress | `false` |

## TLS Certificates

Mount TLS certificates using `extraVolumes` and `extraVolumeMounts`:

```yaml
extraVolumes:
  - name: tls-certs
    secret:
      secretName: sentinel-tls

extraVolumeMounts:
  - name: tls-certs
    mountPath: /etc/sentinel/certs
    readOnly: true
```

Then reference in your configuration:

```yaml
config:
  raw: |
    listeners {
        listener "https" {
            address "0.0.0.0:443"
            protocol "https"
            tls {
                cert "/etc/sentinel/certs/tls.crt"
                key "/etc/sentinel/certs/tls.key"
            }
        }
    }
```

## Related

- [Sentinel](https://github.com/raskell-io/sentinel) - Main proxy repository
- [Documentation](https://sentinel.raskell.io/docs) - Full documentation
- [Sentinel Agent SDK](https://github.com/raskell-io/sentinel-agent-sdk) - Build custom agents

## License

Apache-2.0
