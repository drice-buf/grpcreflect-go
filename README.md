connect-grpcreflect-go
======================

[![Build](https://connectrpc.com/grpcreflect/actions/workflows/ci.yaml/badge.svg?branch=main)](https://connectrpc.com/grpcreflect/actions/workflows/ci.yaml)
[![Report Card](https://goreportcard.com/badge/connectrpc.com/grpcreflect)](https://goreportcard.com/report/connectrpc.com/grpcreflect)
[![GoDoc](https://pkg.go.dev/badge/connectrpc.com/grpcreflect.svg)](https://pkg.go.dev/connectrpc.com/grpcreflect)

`connect-grpcreflect-go` adds support for gRPC's server reflection API to any
`net/http` server &mdash; including those built with [Connect][docs]. With
server reflection enabled, ad-hoc debugging tools can call your gRPC-compatible
handlers and print the responses *without* a copy of the schema.

The exposed reflection API is wire compatible with Google's gRPC
implementations, so it works with [grpcurl], [grpcui], [BloomRPC], and many
other tools.

## Example

```go
package main

import (
  "net/http"

  "golang.org/x/net/http2"
  "golang.org/x/net/http2/h2c"
  grpcreflect "connectrpc.com/grpcreflect"
)

func main() {
  mux := http.NewServeMux()
  reflector := grpcreflect.NewStaticReflector(
    "acme.user.v1.UserService",
    "acme.group.v1.GroupService",
    // protoc-gen-connect-go generates package-level constants
    // for these fully-qualified protobuf service names, so you'd more likely
    // reference userv1.UserServiceName and groupv1.GroupServiceName.
  )
  mux.Handle(grpcreflect.NewHandlerV1(reflector))
  // Many tools still expect the older version of the server reflection API, so
  // most servers should mount both handlers.
  mux.Handle(grpcreflect.NewHandlerV1Alpha(reflector))
  // If you don't need to support HTTP/2 without TLS (h2c), you can drop
  // x/net/http2 and use http.ListenAndServeTLS instead.
  http.ListenAndServe(
    ":8080",
    h2c.NewHandler(mux, &http2.Server{}),
  )
}
```

## Status

Like [`connect-go`][connect], `connect-grpcreflect-go` is a release
candidate. We plan to tag further release candidates as necessary and a stable
v1 soon after the Go 1.19 release.

## Support and Versioning

`connect-grpcreflect-go` supports:

* The [two most recent major releases][go-support-policy] of Go, with a minimum
  of Go 1.18.
* [APIv2][] of protocol buffers in Go (`google.golang.org/protobuf`).

Within those parameters, it follows semantic versioning.

## Legal

Offered under the [Apache 2 license][license].

[APIv2]: https://blog.golang.org/protobuf-apiv2
[BloomRPC]: https://github.com/bloomrpc/bloomrpc
[connect]: https://connectrpc.com/connect
[docs]: https://connectrpc.com
[go-support-policy]: https://golang.org/doc/devel/release#policy
[grpcui]: https://github.com/fullstorydev/grpcui
[grpcurl]: https://github.com/fullstorydev/grpcurl
[license]: https://connectrpc.com/grpcreflect/blob/main/LICENSE.txt
