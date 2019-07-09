# Riptide 3.0 Migration Guide

## Before You Start

First, **Riptide 3.0 requires Spring Boot 2.0**. Spring Boot 1.x is no longer supported. Neither is Spring 4.

If you are running Spring Boot 1.x with Spring 4, consider using [Riptide 2.x](https://github.com/zalando/riptide/releases/tag/2.11.0).

Modules have been added/removed/changed and the configuration structure in your `application.yml` will most like 
require change. Please read the following sections carefully:

## New Features

- [Authorization](riptide-auth)
- Caching
- [Chaos and Fault Injection](riptide-chaos)
- [Async/RestTemplate Compatibility](riptide-compatibility)
- [Idempotency Detection](riptide-idempotency)
- [Native Logbook support](riptide-logbook)
- [OpenTracing](riptide-opentracing)
- [SOAP](riptide-soap)

## Async/RestTemplate support

Riptide 2.x had a
[`PluginInterceptor`](https://github.com/zalando/riptide/blob/2.11.0/riptide-core/src/main/java/org/zalando/riptide/PluginInterceptor.java)
which allowed to use a Riptide `Plugin` in an ordinary `RestTemplate` or `AsyncRestTemplate`. The Riptide Spring Boot
Auto Configuration also made use of that and registered both kinds of templates for each configured client, allowing
to inject them instead of an `Http` instance:

```java
@Autowired
Client(RestTemplate example) {
    // ...
}
```

Both, the `PluginInterceptor` as well as pre-configured templates have been removed. As a cleaner alternative Riptide
now offers a [Compatibility](riptide-compatibility) module which includes custom implementations of Spring's
`RestOperations` and `AsyncRestOperations` which use Riptide under the hood. The Auto Configuration registers an
instance of both automatically:

```java
@Autowired
Client(RestOperations example) {
    // ...
}
```

## Failsafe

**Riptide now requires Failsafe 2.x**

## Logbook

**Riptide now requires Logbook 2.0**

Riptide used to rely on Logbook's integration for the Apache HTTP Client. The usages of the 
`LogbookHttpRequestInterceptor` and `LogbookHttpResponseInterceptor` have been replaced by a
[`LogbookPlugin`](riptide-logbook/src/main/java/org/zalando/riptide/logbook/LogbookPlugin.java).

## Micrometer

Module, package and plugin was renamed:

- `riptide-metrics` is now `riptide-micrometer`
- `org.zalando.riptide.metrics` is now `org.zalando.riptide.micrometer`
- `MetricsPlugin` is now `MicrometerPlugin`

## OAuth

STUPS OAuth 2.0 Token support has been dropped in favor of
[K8s](https://kubernetes-on-aws.readthedocs.io/en/latest/user-guide/zalando-iam.html).

## Request compression

The Apache HTTP client specific `GzipHttpRequestInterceptor` has been replaced with a request factory agnostic 
[`RequestCompressionPlugin`](riptide-core/src/main/java/org/zalando/riptide/RequestCompressionPlugin.java). 

## Spring Boot Auto Configuration

### Added `enabled` properties

:warning: **All nested configurations now have an `enabled` flag which is disabled by default**

  - added `riptide.defaults.backup-request.enabled` (default: `false`)
  - added `riptide.defaults.certificate-pinning.enabled` (default: `false`)
  - added `riptide.defaults.circuit-breaker.enabled` (default: `false`)
  - added `riptide.defaults.logging.enabled` (default: `false`️)
  - added `riptide.defaults.metrics.enabled` (default: `false`)
  - added `riptide.defaults.oauth.enabled` (default: `false`)
  - added `riptide.defaults.request-compression.enabled` (default: `false`)
  - added `riptide.defaults.retry.enabled` (default: `false`)
  - added `riptide.defaults.retry.backoff.enabled` (default: `false`)
  - added `riptide.defaults.stack-trace-preservation.enabled` (default: `false`)
  - added `riptide.defaults.timeouts.enabled` (default: `false`)
  - added `riptide.defaults.transient-fault-detection.enabled` (default: `false`)

### Renamed properties

| Before                                           | After                                                    |
|--------------------------------------------------|----------------------------------------------------------|
| `riptide.defaults.keystore.password`             | `riptide.defaults.certificate-pinning.keystore.password` |
| `riptide.defaults.keystore.path`                 | `riptide.defaults.certificate-pinning.keystore.path`     |
| `riptide.defaults.connect-timeout`               | `riptide.defaults.connections.connect-timeout`           |
| `riptide.defaults.max-connections-per-route`     | `riptide.defaults.connections.max-per-route`             |
| `riptide.defaults.max-connections-total`         | `riptide.defaults.connections.max-total`                 |
| `riptide.defaults.socket-timeout`                | `riptide.defaults.connections.socket-timeout`            |
| `riptide.defaults.connection-time-to-live`       | `riptide.defaults.connections.time-to-live`              |
| `riptide.defaults.record-metrics`                | `riptide.defaults.metrics.enabled`                       |
| `riptide.defaults.compress-request`              | `riptide.defaults.request-compression.enabled`           |
| `riptide.defaults.preserve-stack-trace`          | `riptide.defaults.stack-trace-preservation.enabled`      |
| `riptide.defaults.thread-pool.keep-alive`        | `riptide.defaults.threads.keep-alive`                    |
| `riptide.defaults.thread-pool.max-size`          | `riptide.defaults.threads.max-size`                      |
| `riptide.defaults.thread-pool.min-size`          | `riptide.defaults.threads.min-size`                      |
| `riptide.defaults.thread-pool.queue-size`        | `riptide.defaults.threads.queue-size`                    |
| `riptide.defaults.timeout`                       | `riptide.defaults.timeouts.global`                       |
| `riptide.defaults.detect-transient-faults`       | `riptide.defaults.transient-fault-detection.enabled`     |
| `riptide.clients.<id>.keystore.password`         | `riptide.clients.<id>.certificate-pinning.keystore.password` |
| `riptide.clients.<id>.keystore.path`             | `riptide.clients.<id>.certificate-pinning.keystore.path`     |
| `riptide.clients.<id>.connect-timeout`           | `riptide.clients.<id>.connections.connect-timeout`           |
| `riptide.clients.<id>.max-connections-per-route` | `riptide.clients.<id>.connections.max-per-route`             |
| `riptide.clients.<id>.max-connections-total`     | `riptide.clients.<id>.connections.max-total`                 |
| `riptide.clients.<id>.socket-timeout`            | `riptide.clients.<id>.connections.socket-timeout`            |
| `riptide.clients.<id>.connection-time-to-live`   | `riptide.clients.<id>.connections.time-to-live`              |
| `riptide.clients.<id>.record-metrics`            | `riptide.clients.<id>.metrics.enabled`                       |
| `riptide.clients.<id>.compress-request`          | `riptide.clients.<id>.request-compression.enabled`           |
| `riptide.clients.<id>.preserve-stack-trace`      | `riptide.clients.<id>.stack-trace-preservation.enabled`      |
| `riptide.clients.<id>.thread-pool.keep-alive`    | `riptide.clients.<id>.threads.keep-alive`                    |
| `riptide.clients.<id>.thread-pool.max-size`      | `riptide.clients.<id>.threads.max-size`                      |
| `riptide.clients.<id>.thread-pool.min-size`      | `riptide.clients.<id>.threads.min-size`                      |
| `riptide.clients.<id>.thread-pool.queue-size`    | `riptide.clients.<id>.threads.queue-size`                    |
| `riptide.clients.<id>.timeout`                   | `riptide.clients.<id>.timeouts.global`                       |
| `riptide.clients.<id>.detect-transient-faults`   | `riptide.clients.<id>.transient-fault-detection.enabled`     |

### Removed properties

  - removed `riptide.defaults.oauth.access-token-url`
  - removed `riptide.defaults.oauth.scheduling-period`
  - removed `riptide.defaults.oauth.connect-timeout`
  - removed `riptide.defaults.oauth.socket-timeout`
  - removed `riptide.clients.<id>.oauth.access-token-url`
  - removed `riptide.clients.<id>.oauth.scheduling-period`
  - removed `riptide.clients.<id>.oauth.connect-timeout`
  - removed `riptide.clients.<id>.oauth.socket-timeout`

## Tracer

**Riptide now requires Tracer 2.0**, which in turn is built in top of and therefore **requires OpenTracing.**

Riptide used to rely on Tracer's integration for the Apache HTTP Client. The usages of the
`TracerHttpRequestInterceptor` have been replaced partially by the new 
[`OpenTracingPlugin`](riptide-opentracing/src/main/java/org/zalando/riptide/opentracing/OpenTracingPlugin.java) and
Tracer's new `FlowHttpRequestInterceptor`.