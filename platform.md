# Platform Interface Specification

This document specifies the interface between a lifecycle and a platform.

A platform orchestrates a lifecycle to make buildpack functionality available to end-users such as application developers.

Examples of a platform might include:

1. A local CLI tool that uses buildpacks to create OCI images
2. A plugin for a continuous integration service that uses buildpacks to create OCI images
3. A cloud application platform that uses buildpacks to build source code before deployment

## Table of Contents

1. [Stacks](#stacks)
   1. [Compatibility Guarantees](#compatibility-guarantees)
   2. [Build Image](#build-image)
   3. [Run Image](#run-image)
2. [Buildpacks](#buildpacks)
   1. [Buildpacks Directory Layout](#buildpacks-directory-layout)
3. [Security Considerations](#security-considerations)
4. [Additional Guidance](#additional-guidance)
   1. [Environment](#environment)
   2. [Run Image Rebasing](#run-image-rebasing)
   3. [Caching](#caching)
5. [Data Format](#data-format)
   1. [order.toml (TOML)](#order.toml-(toml))
   2. [group.toml (TOML)](#group.toml-(toml))

## Stacks

A **stack** is a contract defined by a base run OCI image and a base build OCI image that are only updated with security patches.
Stack images can be modified with mixins in order to make additive changes to the contract.

A **mixin** is a named set of additions to a stack that avoid changing the behavior of buildpacks or apps that do not depend on the mixin.

A **launch layer** refers to a layer in the app OCI image created from a  `<layers>/<layer>` directory as specified in the [Buildpack Interface Specification](buildpack.md).

An **app layer** refers to a layer created from the `<app>` directory as specified in the [Buildpack Interface Specification](buildpack.md).

### Compatibility Guarantees

Stack image authors SHOULD ensure that build image versions maintain [ABI-compatibility](https://en.wikipedia.org/wiki/Application_binary_interface) with previous versions, although violating this requirement will not change the behavior of previously built images containing app and launch layers.

Stack image authors MUST ensure that new run image versions maintain [ABI-compatibility](https://en.wikipedia.org/wiki/Application_binary_interface) with previous versions.
Stack image authors MUST ensure that app and launch layers do not change behavior when the run image layers are upgraded to newer versions, unless those behavior changes are intended to fix security vulnerabilities.

Mixin authors MUST ensure that mixins do not affect the [ABI-compatibility](https://en.wikipedia.org/wiki/Application_binary_interface) of any object code compiled to run on the base stack images without mixins.

During build, platforms MUST use the same set of mixins for the run image as were used in the build image (excluding mixins that have a stage specifier).

### Build Image

The platform MUST execute the detection and build phases of the lifecycle on the build image.

The platform MUST ensure that:

- The image config's `User` field is set to a non-root user with a writable home directory.
- The image config's `Env` field has the environment variable `CNB_STACK_ID` set to the stack ID.
- The image config's `Env` field has the environment variable `CNB_USER_ID` set to the user [†](README.md#linux-only)UID/[‡](README.md#windows-only)SID of the user specified in the `User` field.
- The image config's `Env` field has the environment variable `CNB_GROUP_ID` set to the primary group [†](README.md#linux-only)GID/[‡](README.md#windows-only)SID of the user specified in the `User` field.
- The image config's `Label` field has the label `io.buildpacks.stack.id` set to the stack ID.
- The image config's `Label` field has the label `io.buildpacks.stack.mixins` set to a JSON array containing mixin names for each mixin applied to the image.

To initiate the detection phase, the platform MUST invoke the `/cnb/lifecycle/detector` executable with the user and environment defined in the build image config.
Invoking this executable with no flags is equivalent to the following invocation including all accepted flags and their default values.

```bash
/cnb/lifecycle/detector -buildpacks /cnb/by-id -order /cnb/order.toml -group ./group.toml -plan ./plan.toml
```

Where:

- `-buildpacks` MUST specify input from a buildpacks directory as defined in the [Buildpacks Directory Layout](#buildpacks-directory-layout) section.
- `-order` MUST specify input from an overriding `order.toml` file path as defined in the [Data Format](#data-format) section.
- `-group` MUST specify output to a `group.toml` file path as defined in the [Data Format](#data-format) section.
- `-plan` MUST specify output to a Build Plan as defined in the [Buildpack Interface Specification](buildpack.md).

To initiate the build phase, the platform MUST invoke the `/cnb/lifecycle/builder` executable with the user and environment defined in the build image config.
Invoking this executable with no flags is equivalent to the following invocation including all accepted flags and their default values.

```bash
/cnb/lifecycle/builder -buildpacks /cnb/by-id -group ./group.toml -plan ./plan.toml
```

Where:

- `-buildpacks` MUST specify input from a buildpacks directory as defined in the [Buildpacks Directory Layout](#buildpacks-directory-layout) section.
- `-group` MUST specify input from a `group.toml` file path as defined in the [Data Format](#data-format) section.
- `-plan` MUST specify input from a Build Plan as defined in the [Buildpack Interface Specification](buildpack.md).

### Run Image

The platform MUST provide the lifecycle with a reference to the run image during the export phase.

The platform MUST ensure that:

- The image config's `User` field is set to a user with the same user [†](README.md#linux-only)UID/[‡](README.md#windows-only)SID and primary group [†](README.md#linux-only)GID/[‡](README.md#windows-only)SID as in the build image.
- The image config's `Label` field has the label `io.buildpacks.stack.id` set to the stack ID.
- The image config's `Label` field has the label `io.buildpacks.stack.mixins` set to a JSON array containing mixin names for each mixin applied to the image.

### Mixins

A mixin name MUST only be defined by the author of its corresponding stack.
A mixin name MUST always be used to specify the same set of changes.
A mixin name MUST only contain a `:` character as part of an optional stage specifier.

A mixin prefixed with the `build:` stage specifier only affects the build image and does not need to be specified on the run image.
A mixin prefixed with the `run:` stage specifier only affects the run image and does not need to be specified on the build image.

A platform MAY support any number of mixins for a given stack in order to support application code or buildpacks that require those mixins.

Changes introduced by mixins SHOULD be restricted to the addition of operating system software packages that are regularly patched with strictly backwards-compatible security fixes.
However, mixins MAY consist of any changes that follow the [Compatibility Guarantees](#compatibility-guarantees).

## Buildpacks

### Buildpacks Directory Layout

The buildpacks directory MUST contain unarchived buildpacks such that:

- Each top-level directory is a buildpack ID.
- Each second-level directory is a buildpack version.

## Security Considerations

The platform SHOULD run each phase of the lifecycle in an isolated container to prevent untrusted app and buildpack code from accessing storage credentials needed during the export and analysis phases.
A more thorough explanation is provided in the [Buildpack Interface Specification](buildpack.md).

## Additional Guidance

### Environment

User-provided environment variables intended for build and launch SHOULD NOT come from the same list.
The end-user SHOULD be encouraged to define them separately.
The platform MAY determine the initial environment of the build phase, detection phase, and launch.
The lifecycle MUST NOT assume that all platforms provide an identical environment.

### Run Image Rebasing

Run image rebasing allows for fast stack updates for already-exported OCI images with minimal data transfer when those images are stored on a Docker registry.
When a new stack version with the same stack ID is available, the app layers and launch layers SHOULD be rebased on the new run image by updating the image's configuration to point at the new run image.
Once the new run image is present on the registry, filesystem layers SHOULD NOT be uploaded or downloaded.

The new run image MUST have an identical stack ID and MUST include the exact same set of mixins.

![Launch](img/launch.svg)

### Caching

Each platform SHOULD implement caching so as to appropriately optimize performance.
Cache locality and availability MAY vary between platforms.

## Data Format

### order.toml (TOML)

```toml
[[order]]
[[order.group]]
id = "<buildpack ID>"
version = "<buildpack version>"
optional = false
```

Where:

- Both `id` and `version` MUST be present for each buildpack object in a group.
- The value of `optional` MUST default to false if not specified.

### group.toml (TOML)

```toml
group = [
  { id = "<buildpack ID>", version = "<buildpack version>" }
]
```

Where:

- Both `id` and `version` MUST be present for each buildpack object in a group.
