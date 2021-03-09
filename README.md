# InfluxDB [![CircleCI](https://circleci.com/gh/influxdata/influxdb.svg?style=svg)](https://circleci.com/gh/influxdata/influxdb)
[![Slack Status](https://img.shields.io/badge/slack-join_chat-white.svg?logo=slack&style=social)](https://www.influxdata.com/slack)

InfluxDB is an open source time series platform. This includes APIs for storing and querying data, processing it in the background for ETL or monitoring and alerting purposes, user dashboards, and visualizing and exploring the data and more. The master branch on this repo now represents the latest InfluxDB, which now includes functionality for Kapacitor (background processing) and Chronograf (the UI) all in a single binary.

The list of InfluxDB Client Libraries that are compatible with the latest version can be found in [our documentation](https://docs.influxdata.com/influxdb/latest/tools/client-libraries/).

If you are looking for the 1.x line of releases, there are branches for each minor version as well as a `master-1.x` branch that will contain the code for the next 1.x release. The master-1.x [working branch is here](https://github.com/influxdata/influxdb/tree/master-1.x). The [InfluxDB 1.x Go Client can be found here](https://github.com/influxdata/influxdb1-client).

## Installing from Source

We have nightly and versioned Docker images, Debian packages, RPM packages, and tarballs of InfluxDB available at the [InfluxData downloads page](https://portal.influxdata.com/downloads/). We also provide the `influx` command line interface (CLI) client as a separate binary available at the same location.

## Building From Source

This project requires Go 1.13 and Go module support.

Set `GO111MODULE=on` or build the project outside of your `GOPATH` for it to succeed.

The project also requires a recent stable version of Rust. We recommend using [rustup](https://rustup.rs/) to install Rust.

If you are getting an `error loading module requirements` error with `bzr executable file not found in $PATH”` on `make`, then you need to ensure you have `bazaar`, `protobuf`, and `yarn` installed.

- OSX: `brew install bazaar protobuf yarn`
- Linux (Arch): `pacman -S bzr protobuf yarn`
- Linux (Ubuntu): `apt install bzr protobuf-compiler libprotobuf-dev yarnpkg`

**NB:** For RedHat, there are some extra steps:

1. You must enable the [EPEL](https://fedoraproject.org/wiki/EPEL)
2. You must add the `yarn` [repository](https://yarnpkg.com/lang/en/docs/install/#centos-stable)

For information about modules, please refer to the [wiki](https://github.com/golang/go/wiki/Modules).

A successful `make` run results in two binaries, with platform-dependent paths:

```
$ make
...
env GO111MODULE=on go build -tags 'assets ' -o bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx ./cmd/influx
env GO111MODULE=on go build -tags 'assets ' -o bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influxd ./cmd/influxd
```

`influxd` is the InfluxDB service.
`influx` is the CLI management tool.

Start the service.
Logs to stdout by default:

```
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influxd
```

### Building with the go command

The `Makefile` provides a wrapper around configuring the utilities for building influxdb. For those wanting to use the `go` command directly, one of two things can be done.

First, the `env` script is located in the root of the repository. This script can be used to execute `go` commands with the appropriate environment configuration.

```bash
$ ./env go build ./cmd/influxd
```

Another method is to configure the `pkg-config` utility. Follow the instructions [here](https://github.com/influxdata/flux#getting-started) to install and configure `pkg-config` and then the normal go commands will work.

The first step is to install the `pkg-config` command.

```bash
# On Debian/Ubuntu
$ sudo apt-get install -y clang pkg-config
# On Mac OS X with Homebrew
$ brew install pkg-config
```

Install the `pkg-config` wrapper utility of the same name to a different path that is earlier in the PATH.

```bash
# Install the pkg-config wrapper utility
$ go build -o ~/go/bin/ github.com/influxdata/pkg-config
# Ensure the GOBIN directory is on your PATH
$ export PATH=$HOME/go/bin:${PATH}
$ which -a pkg-config
/home/user/go/bin/pkg-config
/usr/bin/pkg-config
```

Then all `go` build commands should work.

```bash
$ go build ./cmd/influxd
$ go test ./...
```

## Getting Started

For a complete getting started guide, please see our full [online documentation site](https://docs.influxdata.com/influxdb/latest/). 

To write and query data or use the API in any way, you'll need to first create a user, credentials, organization and bucket.
Everything in InfluxDB is organized under a concept of an organization. The API is designed to be multi-tenant.
Buckets represent where you store time series data.
They're synonymous with what was previously in InfluxDB 1.x a database and retention policy.

The simplest way to get set up is to point your browser to [http://localhost:8086](http://localhost:8086) and go through the prompts.

You can also get set up from the CLI using the command `influx setup`:


```bash
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx setup
Welcome to InfluxDB 2.0!
Please type your primary username: marty

Please type your password: 

Please type your password again: 

Please type your primary organization name.: InfluxData

Please type your primary bucket name.: telegraf

Please type your retention period in hours.
Or press ENTER for infinite.: 72


You have entered:
  Username:          marty
  Organization:      InfluxData
  Bucket:            telegraf
  Retention Period:  72 hrs
Confirm? (y/n): y

UserID                  Username        Organization    Bucket
033a3f2c5ccaa000        marty           InfluxData      Telegraf
Your token has been stored in /Users/marty/.influxdbv2/credentials
```

You can run this command non-interactively using the `-f, --force` flag if you are automating the setup.
Some added flags can help:
```bash
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx setup \
--username marty \
--password F1uxKapacit0r85 \
--org InfluxData \
--bucket telegraf \
--retention 168 \
--token where-were-going-we-dont-need-roads \
--force
```

Once setup is complete, a configuration profile is created to allow you to interact with your local InfluxDB without passing in credentials each time. You can list and manage those profiles using the `influx config` command.
```bash
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx config
Active	Name	URL			            Org
*	    default	http://localhost:8086	InfluxData
```

## Writing Data
Write to measurement `m`, with tag `v=2`, in bucket `telegraf`, which belongs to organization `InfluxData`:

```bash
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx write --bucket telegraf --precision s "m v=2 $(date +%s)"
```

Since you have a default profile set up, you can omit the Organization and Token from the command.

Write the same point using `curl`:

```bash
curl --header "Authorization: Token $(bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx auth list --json | jq -r '.[0].token')" \
--data-raw "m v=2 $(date +%s)" \
"http://localhost:8086/api/v2/write?org=InfluxData&bucket=telegraf&precision=s"
```

Read that back with a simple Flux query:

```bash
$ bin/$(uname -s | tr '[:upper:]' '[:lower:]')/influx query 'from(bucket:"telegraf") |> range(start:-1h)'
Result: _result
Table: keys: [_start, _stop, _field, _measurement]
                   _start:time                      _stop:time           _field:string     _measurement:string                      _time:time                  _value:float
------------------------------  ------------------------------  ----------------------  ----------------------  ------------------------------  ----------------------------
2019-12-30T22:19:39.043918000Z  2019-12-30T23:19:39.043918000Z                       v                       m  2019-12-30T23:17:02.000000000Z                             2
```

Use the `-r, --raw` option to return the raw flux response from the query. This is useful for moving data from one instance to another as the `influx write` command can accept the Flux response using the `--format csv` option. 

## Introducing Flux

Flux is an MIT-licensed data scripting language (previously named IFQL) used for querying time series data from InfluxDB. The source for Flux is [available on GitHub](https://github.com/influxdata/flux). Learn more about Flux from [CTO Paul Dix's presentation](https://speakerdeck.com/pauldix/flux-number-fluxlang-a-new-time-series-data-scripting-language).

## Contributing to the Project

InfluxDB is an [MIT licensed](LICENSE) open source project and we love our community. The fastest way to get something fixed is to open a PR. Check out our [contributing](CONTRIBUTING.md) guide if you're interested in helping out. Also, join us on our [Community Slack Workspace](https://influxdata.com/slack) if you have questions or comments for our engineering teams.

## CI and Static Analysis

### CI

All pull requests will run through CI, which is currently hosted by Circle.
Community contributors should be able to see the outcome of this process by looking at the checks on their PR.
Please fix any issues to ensure a prompt review from members of the team.

The InfluxDB project is used internally in a number of proprietary InfluxData products, and as such, PRs and changes need to be tested internally.
This can take some time, and is not really visible to community contributors.

### Static Analysis

This project uses the following static analysis tools.
Failure during the running of any of these tools results in a failed build.
Generally, code must be adjusted to satisfy these tools, though there are exceptions.

- [go vet](https://golang.org/cmd/vet/) checks for Go code that should be considered incorrect.
- [go fmt](https://golang.org/cmd/gofmt/) checks that Go code is correctly formatted.
- [go mod tidy](https://tip.golang.org/cmd/go/#hdr-Add_missing_and_remove_unused_modules) ensures that the source code and go.mod agree.
- [staticcheck](http://next.staticcheck.io/docs/) checks for things like: unused code, code that can be simplified, code that is incorrect and code that will have performance issues.

### staticcheck

If your PR fails `staticcheck` it is easy to dig into why it failed, and also to fix the problem.
First, take a look at the error message in Circle under the `staticcheck` build section, e.g.,

```
tsdb/tsm1/encoding.gen.go:1445:24: func BooleanValues.assertOrdered is unused (U1000)
tsdb/tsm1/encoding.go:172:7: receiver name should not be an underscore, omit the name if it is unused (ST1006)
```

Next, go and take a [look here](http://next.staticcheck.io/docs/checks) for some clarification on the error code that you have received, e.g., `U1000`.
The docs will tell you what's wrong, and often what you need to do to fix the issue.

#### Generated Code

Sometimes generated code will contain unused code or occasionally that will fail a different check.
`staticcheck` allows for [entire files](http://next.staticcheck.io/docs/#ignoring-problems) to be ignored, though it's not ideal.
A linter directive, in the form of a comment, must be placed within the generated file.
This is problematic because it will be erased if the file is re-generated.
Until a better solution comes about, below is the list of generated files that need an ignores comment.
If you re-generate a file and find that `staticcheck` has failed, please see this list below for what you need to put back:

|          File          |                             Comment                              |
| :--------------------: | :--------------------------------------------------------------: |
| query/promql/promql.go | //lint:file-ignore SA6001 Ignore all unused code, it's generated |

#### End-to-End Tests

CI also runs end-to-end tests. These test the integration between the influx server the ui. You can run them locally in two steps:

- Start the server in "testing mode" by running `make run-e2e`.
- Run the tests with `make e2e`.
