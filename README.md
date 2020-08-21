# Docker Compose CLI

[![Actions Status](https://github.com/docker/compose-cli/workflows/Continuous%20integration/badge.svg)](https://github.com/docker/compose-cli/actions)

This CLI tool makes it easy to run containers in the cloud using either Amazon
Elastic Container Service
([ECS](https://aws.amazon.com/ecs))
or Microsoft Azure Container Instances
([ACI](https://azure.microsoft.com/services/container-instances))
using the Docker commands you already know.

To get started, all you need is:
* An [AWS](https://aws.amazon.com) or [Azure](https://azure.microsoft.com)
  account
* Windows: Edge release of
  [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
* macOS: The Edge release of
  [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
* Linux:
  TODO <!-- Move instructions from https://github.com/docker/aci-integration-beta to this repo-->

:warning: *This CLI is currently in beta please create*
*[issues](https://github.com/docker/compose-cli/issues) to leave feedback*

## Examples

* ECS: [Deploying Wordpress to the cloud](https://www.docker.com/blog/deploying-wordpress-to-the-cloud/)
* ACI: [Deploying a Minecraft server to the cloud](https://www.docker.com/blog/deploying-a-minecraft-docker-server-to-the-cloud/)

## Developing

### Prerequisites

* Windows:
  * [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-windows)
  * make
* macOS:
  * [Docker Desktop](https://hub.docker.com/editions/community/docker-ce-desktop-mac)
  * make
* Linux:
  * [Docker 19.03 or later](https://docs.docker.com/engine/install/)
  * make

### Building the CLI

Once you have the prerequisites installed, you can build the CLI using:

```console
make
```

This will output a CLI for your host machine in `./bin`.

You will then need to make sure that you have the existing Docker CLI in your
`PATH` with the name `com.docker.cli`. A make target is provided to help with
this:

```console
make moby-cli-link
```

This will create a symbolic link from the existing Docker CLI to
`/usr/local/bin` with the name `com.docker.cli`.

You can statically cross compile the CLI for Windows, macOS, and Linux using the
`cross` target.

### Building with specific backends

You can specify which backends are build using the `BUILD_TAGS` variable.
The available backends are:
* `aci`: For ACI support (always built)
* `ecs`: For ECS support (on by default)
* `example`: Testing backend (off by default)
* `local`: Beginnings of a [moby](https://github.com/moby/moby) backend
  (off by default)

If you want the ACI, ECS and example backends, then you can build as follows:

```console
make BUILD_TAGS=example cli
```

### Updating the API code

The API provided by the CLI is defined using protobuf. If you make changes to
the `.proto` files in [`protos/`](./protos), you will need to regenerate the API
code:

```console
make protos
```

### Unit tests

To run all of the unit tests, run:

```console
make test
```

If you need to update a golden file simply do `go test ./... -test.update-golden`.

### End to end tests

#### Local tests

To run the local end to end tests, run:

```console
make e2e-local
```

Note that this requires the CLI to be built and a local Docker Engine to be
running.

#### ACI tests

To run the end to end ACI tests, you will first need to have an Azure account
and have created a service principal. You can create a service principle using
the Azure CLI after you have done a `docker login azure`:

```console
$ docker login azure
$ az ad sp create-for-rbac --name 'MyTestServicePrincipal' --sdk-auth
```

You can then run the ACI tests using the `e2e-aci` target with the various
`AZURE_` environment variables set:

```console
AZURE_TENANT_ID="xxx" AZURE_CLIENT_ID="yyy" AZURE_CLIENT_SECRET="yyy" make e2e-aci
```

Running the ACI tests will override your local login and the service principal
credentials use a token that cannot be refreshed automatically.

*Note:* You will need to rerun `docker login azure` if you would like to use the
CLI after running the ACI tests.

You can also run a single ACI test by specifying the test name with the
`E2E_TEST` variable:
```console
AZURE_TENANT_ID="xxx" AZURE_CLIENT_ID="yyy" AZURE_CLIENT_SECRET="yyy" make E2E_TEST=TestContainerRun e2e-aci
```

#### ECS tests

To run the end to end ECS tests, you will need to have an AWS account and have
credentials for it in the `~/.aws/credentials` file.

You can then use the `e2e-ecs` target:

```console
TEST_AWS_PROFILE=myProfile TEST_AWS_REGION=eu-west-3 make e2e-ecs
```

## Releases

To create a new release:
* Check that the CI is green on the main branch for commit you want to release
* Create a new tag of the form vx.y.z, following existing tags, and push the tag

Pushing the tag will automatically create a new release and make binaries for
Windows, macOS, and Linux available for download on the
[releases page](https://github.com/docker/compose-cli/releases).
