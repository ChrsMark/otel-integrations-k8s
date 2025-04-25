## Integrations receiver along with receiver creator on K8s 

This playground provides a minimal example of how the 
[integrations receiver](https://github.com/elastic/opentelemetry-collector-components/tree/main/receiver/integrationreceiver)
can work along with the
[receiver creator](https://github.com/open-telemetry/opentelemetry-collector-contrib/blob/main/receiver/receivercreator/README.md).
This allows the users to configure the Collector to dynamically discover moving 
workloads and be able to parse their logs based on configurations packages as otel integrations that the integration
receiver can use.

0. Replace the Dockefile and the manifest.yaml files of https://github.com/elastic/opentelemetry-collector-components
   with the ones of this repository so as to build a custom Collector that can run inside a K8s Pod.
1. Build the custom collector image:
```console
GOOS=linux CGO_ENABLED=0 make genelasticcol
CGO_ENABLED=0 make builddocker TAG=oteldev
docker tag elastic-collector-components:oteldev otelcontribcol-dev:latest && kind load docker-image otelcontribcol-dev:latest
```

2. Deploy the intgrations ConfigMap:
```console
k apply -f integrations.yaml
```
3. Deploy the Collector:
```console
helm install collector oci://ghcr.io/open-telemetry/opentelemetry-helm-charts/opentelemetry-collector --version 0.119.1 --values discovery.yaml 
```
4. Deloy a target apache workload
```console
 helm install apache oci://registry-1.docker.io/bitnamicharts/apache --version 11.3.5 --values apache.yaml 
```
5. Check that apache logs are parsed properly based on the pattern the configured integration defines:
```console
2025-04-25T13:29:25.884Z	info	ResourceLog #0
Resource SchemaURL:
Resource attributes:
     -> k8s.container.restart_count: Str(0)
     -> k8s.pod.uid: Str(8142d8f8-af2f-40e8-a71d-85d624e30081)
     -> k8s.container.name: Str(apache)
     -> k8s.namespace.name: Str(default)
     -> k8s.pod.name: Str(apache-6c4ff878b4-ll59z)
     -> container.id: Str(1992112841b890b5ef1581aba4ac3e79c8826129b02bc97d55e646d1e231a948)
     -> container.image.name: Str(docker.io/bitnami/apache:2.4.63-debian-12-r7)
ScopeLogs #0
ScopeLogs SchemaURL:
InstrumentationScope
LogRecord #0
ObservedTimestamp: 2025-04-25 13:29:25.685543554 +0000 UTC
Timestamp: 2025-04-25 13:29:25.64818397 +0000 UTC
SeverityText:
SeverityNumber: Unspecified(0)
Body: Str(10.244.0.1 - - [25/Apr/2025:13:29:25 +0000] "GET / HTTP/1.1" 200 45)
Attributes:
     -> log.iostream: Str(stdout)
     -> http_code: Str(200)
     -> http_size: Str(45)
     -> source_ip: Str(10.244.0.1)
     -> timestamp_log: Str(25/Apr/2025:13:29:25 +0000)
     -> log.file.path: Str(/var/log/pods/default_apache-6c4ff878b4-ll59z_8142d8f8-af2f-40e8-a71d-85d624e30081/apache/0.log)
     -> http_path: Str(/)
     -> logtag: Str(F)
     -> http_version: Str(HTTP/1.1)
     -> http_method: Str(GET)
     -> resource: Str(8142d8f8-af2f-40e8-a71d-85d624e30081)
Trace ID:
Span ID:
Flags: 0
```
