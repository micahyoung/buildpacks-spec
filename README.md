# Buildpack API v3 - Specification

This specification defines interactions between a platform, a lifecycle, a number of buildpacks, and an application
1. For the purpose of transforming that application into an OCI image and
2. For the purpose of developing or executing automated tests on that application.

A **buildpack** is software that partially or completely transforms application source code into runnable artifacts.

A **lifecycle** is software that orchestrates buildpacks and transforms the resulting artifacts into an OCI image.

A **platform** is software that orchestrates a lifecycle to make buildpack functionality available to end-users such as application developers.

## Notational Conventions

### Key Words
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" are to be interpreted as described in [RFC 2119](http://tools.ietf.org/html/rfc2119).

The key words "unspecified", "undefined", and "implementation-defined" are to be interpreted as described in the [rationale for the C99 standard](http://www.open-std.org/jtc1/sc22/wg14/www/C99RationaleV5.10.pdf#page=18).

An implementation is not compliant if it fails to satisfy one or more of the MUST, MUST NOT, REQUIRED, SHALL, or SHALL NOT requirements for the protocols it implements.
An implementation is compliant if it satisfies all the MUST, MUST NOT, REQUIRED, SHALL, and SHALL NOT requirements for the protocols it implements.

### Operating System Conventions

When a word or bullet point is prefixed with a <a name="linux-only">†</a>, it SHALL be assumed to apply only to Linux stacks.

When a word or bullet point is prefixed with a <a name="windows-only">‡</a>, it SHALL be assumed to apply only to Windows stacks.

When the specification denotes a "shell", Linux stacks MUST use the Bourne Again Shell (`bash`) version 3 or greater and Windows stacks MUST use Command Prompt (`cmd.exe`) unless otherwise specified.

When the specification denotes a filesystem path using Unix path notation (e.g. `/cnb/lifecycle`), on Windows stacks this notation SHALL be interpreted to represent a path where all unix file path separators are replaced with the Windows filepath separator (`\`) and absolute paths are assumed to be rooted in the default drive (e.g. `C:\cnb\lifecycle`).

When the specification refers to an executable file with Unix path notation (e.g. `/cnb/lifecycle/builder`), Windows stacks MUST assume two files are intented: one with the suffix  `.exe` (e.g. `C:\cnb\lifecycle\builder.exe`) and another with the suffix `.bat` (e.g. `C:\cnb\lifecycle\builder.bat`).

When the specification refers to a path in the context of OCI layer blob (e.g. `/cnb/buildpacks/<buildpack ID>/<buildpack version>/`)  it SHALL be assumed that this path is prefixed with `Files` (e.g. `Files/cnb/buildpacks/<buildpack ID>/<buildpack version>/`) when the layer blob is associated with a Windows platform OCI image.

## Sections

- [Buildpack Interface Specification](buildpack.md)
- [Platform Interface Specification](platform.md)
- [Distribution Specification](distribution.md)
