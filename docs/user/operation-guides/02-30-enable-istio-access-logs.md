---
title: Enable Istio access logs
---

You can enable [Istio access logs](https://istio.io/latest/docs/tasks/observability/logs/access-log/) to provide fine-grained details about the access to workloads that are part of the Istio service mesh. This can help indicate the four “golden signals” of monitoring (latency, traffic, errors, and saturation) and troubleshooting anomalies.
The Istio setup shipped with Kyma provides a pre-configured [extension provider](https://istio.io/latest/docs/tasks/observability/telemetry) for access logs, which will configure the istio-proxies to print access logs to stdout using JSON format. It uses a configuration similar to the following one:

```yaml
  extensionProviders:
    - name: stdout-json
      envoyFileAccessLog:
        path: "/dev/stdout"
        logFormat:
          labels:
            ...
            traceparent: "%REQ(TRACEPARENT)%"
            tracestate: "%REQ(TRACESTATE)%"
```

The [log format](https://github.com/kyma-project/istio/blob/main/manifests/istio-operator-template.yaml#L160) is based on the Istio default format enhanced with the attributes relevant for identifying the related trace context conform to the [w3c-tracecontext](https://www.w3.org/TR/trace-context/) protocol. See [Kyma tracing](https://kyma-project.io/#/telemetry-manager/user/03-traces) for more details on tracing. See [Istio tracing](https://kyma-project.io/#/telemetry-manager/user/03-traces?id=istio) on how to enable trace context propagation with Istio.

>**CAUTION:** Enabling access logs may drastically increase logs volume and might quickly fill up your log storage. Also, the provided feature uses an API in alpha state, which may change in future releases.

## Configuration

Use the Telemetry API to selectively enable Istio access logs. You can enable access logs for an entire namespace, for a selective workload, or on the Istio Gateway scope.

### Configure Istio Access Logs for the Entire Namespace

1. In the following sample configuration, replace `{YOUR_NAMESPACE}` with your namespace.
2. To apply the configuration, run `kubectl apply`.

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: access-config
  namespace: {YOUR_NAMESPACE}
spec:
  accessLogging:
    - providers:
      - name: stdout-json
```

### Configure Istio Access Logs for a Selective Workload

To configure label-based selection of workloads, use a [selector](https://istio.io/latest/docs/reference/config/type/workload-selector/#WorkloadSelector).
1. In the following sample configuration, replace `{YOUR_NAMESPACE}` and `{YOUR_LABEL}` with your namespace and the label of the workload, respectively.
2. To apply the configuration, run `kubectl apply`.

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: access-config
  namespace: {YOUR_NAMESPACE}
spec:
  selector:
    matchLabels:
      service.istio.io/canonical-name: {YOUR_LABEL}
  accessLogging:
    - providers:
      - name: stdout-json
```

### Configure Istio Access Logs for a Specific Gateway

Instead of enabling the access logs for all the individual proxies of the workloads you have, you can enable the logs for the proxy used by the related Istio Ingress Gateway:

```yaml
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: access-config
  namespace: istio-system
spec:
  selector:
    matchLabels:
      istio: ingressgateway
  accessLogging:
    - providers:
      - name: stdout-json
```