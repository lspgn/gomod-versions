# gomod-versions

Just a small test to observe how go.mods behaves.

This repository contains three tags, v1, v2, and v3. Only v3 has version indication in `go.mod`.
Check these links:
* [Go Modules: v2 and Beyond](https://blog.golang.org/v2-go-modules)
* [Releasing Modules (v2 or Higher)](https://github.com/golang/go/wiki/Modules#releasing-modules-v2-or-higher)

Every test is done from another repository with a blank `go.mod` containing:
```go
module test

go 1.13
```

## v1

This is the default.

```go
package main

import (
	"fmt"
	"github.com/lspgn/gomod-versions/pkg"
)

func main() {
	fmt.Printf("Hello %d\n", pkg.MyVersion())
}
```

Should return
```bash
Hello 0
```

## v2

To get v2, one should add the exact commit in the `go.mod`.
Otherwise the following appears:
```bash
go: finding github.com/lspgn/gomod-versions v2.0.0
go: errors parsing go.mod:
/.../gomod-versions-manual/go.mod:6: require github.com/lspgn/gomod-versions: version "v2.0.0" invalid: module contains a go.mod file, so major version must be compatible: should be v0 or v1, not v2
```

```go
module test

go 1.13

require (
	github.com/lspgn/gomod-versions 0b885af817c3f92354def5a5b6862a86a5773a52
)
```

Then the same code as above will show:

```bash
Hello 2
```

`go.mod` will then display:

```go
require github.com/lspgn/gomod-versions v1.0.1-0.20200302020824-0b885af817c3
```

## v3

```go
package main

import (
	"fmt"
	"github.com/lspgn/gomod-versions/v3/pkg"
	v1 "github.com/lspgn/gomod-versions/pkg"
)

func main() {
	fmt.Printf("Hello %d\n", pkg.MyVersion())
	fmt.Printf("Hello %d\n", v1.MyVersion())
}
```

Should return
```bash
Hello 3
Hello 0
```

## v4

Unfortunately, it is not possible to use v4 from a subdirectory as the root `go.mod` contains v3.

```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/v4/@v/list
not found: github.com/lspgn/gomod-versions/v4@v4.0.0: invalid version: go.mod has non-.../v4 module path "github.com/lspgn/gomod-versions/v3" (and .../v4/go.mod does not exist) at revision v4.0.0
```

# Effects on proxy

The following command: `curl --silent https://github.com/lspgn/gomod-versions | grep go-import`
returns
```html
<meta name="go-import" content="github.com/lspgn/gomod-versions git https://github.com/lspgn/gomod-versions.git">
```

If we use [Go Proxy](https://golang.org/cmd/go/#hdr-Module_proxy_protocol):

By default:
```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/@v/list
v1.0.0
```
If we pass v3:
```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/v3/@v/list
v3.0.0
```
v2 does not exists:
```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/v2/@v/list
not found: github.com/lspgn/gomod-versions/v2@v2.0.0: invalid version: go.mod has non-.../v2 module path "github.com/lspgn/gomod-versions" (and .../v2/go.mod does not exist) at revision v2.0.
```
Pass directly the commit of v2:
```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/@v/0b885af817c3.info
{"Version":"v1.0.1-0.20200302020824-0b885af817c3","Time":"2020-03-02T02:08:24Z"
```
Result can be used to fetch [specific documentation](https://pkg.go.dev/github.com/lspgn/gomod-versions@v1.0.1-0.20200302020824-0b885af817c3?tab=overview).

We can get the `go.mod` associated with a specific version.

```bash
$ curl https://proxy.golang.org/github.com/lspgn/gomod-versions/v3/@v/v3.0.0.mod
module github.com/lspgn/gomod-versions/v3

go 1.13
```
