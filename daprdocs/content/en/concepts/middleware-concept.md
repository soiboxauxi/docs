---
type: docs
title: "Middleware pipelines"
linkTitle: "Middleware"
weight: 400
description: "Custom processing pipelines of chained middleware components"
---

Dapr allows custom processing pipelines to be defined by chaining a series of middleware components. A request goes through all defined middleware components before it's routed to user code, and then goes through the defined middleware, in reverse order, before it's returned to the client, as shown in the following diagram.

<img src="/images/middleware.png" width=400>

## Customize processing pipeline

When launched, a Dapr sidecar constructs a middleware processing pipeline. By default the pipeline consists of [tracing middleware]({{< ref tracing.md >}}) and CORS middleware. Additional middleware, configured by a Dapr [configuration]({{< ref configuration-concept.md >}}), can be added to the pipeline in the order they are defined. The pipeline applies to all Dapr API endpoints, including state, pub/sub, service invocation, bindings, security and others.

> **NOTE:** Dapr provides a **middleware.http.uppercase** pre-registered component that changes all text in a request body to uppercase. You can use it to test/verify if your custom pipeline is in place.

The following configuration example defines a custom pipeline that uses a [OAuth 2.0 middleware]({{< ref oauth.md >}}) and an uppercase middleware component. In this case, all requests are authorized through the OAuth 2.0 protocol, and transformed to uppercase text, before they are forwarded to user code.

```yaml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: pipeline
  namespace: default
spec:
  httpPipeline:
    handlers:
    - name: oauth2
      type: middleware.http.oauth2
    - name: uppercase
      type: middleware.http.uppercase
```

## Writing a custom middleware

Dapr uses [FastHTTP](https://github.com/valyala/fasthttp) to implement it's HTTP server. Hence, your HTTP middleware needs to be written as a FastHTTP handler. Your middleware needs to implement a middleware interface, which defines a **GetHandler** method that returns a **fasthttp.RequestHandler**:

```go
type Middleware interface {
  GetHandler(metadata Metadata) (func(h fasthttp.RequestHandler) fasthttp.RequestHandler, error)
}
```

Your handler implementation can include any inbound logic, outbound logic, or both:

```go
func GetHandler(metadata Metadata) fasthttp.RequestHandler {
  return func(h fasthttp.RequestHandler) fasthttp.RequestHandler {
    return func(ctx *fasthttp.RequestCtx) {
      // inboud logic
      h(ctx)  // call the downstream handler
      // outbound logic
    }
  }
}
```

## Adding new middleware components
Your middleware component can be contributed to the [components-contrib repository](https://github.com/dapr/components-contrib/tree/master/middleware). 

Then submit another pull request against the [Dapr runtime repository](https://github.com/dapr/dapr) to register the new middleware type. You'll need to modify the **Load()** method in [registry.go]( https://github.com/dapr/dapr/blob/master/pkg/components/middleware/http/registry.go) to register your middleware using the **Register** method.

## Next steps
* [How-To: Configure API authorization with OAuth]({{< ref oauth.md >}})
