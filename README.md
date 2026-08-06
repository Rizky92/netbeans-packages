# Apache NetBeans package builds

This repository contains a configuration and build system for packages of
[Apache NetBeans](https://netbeans.apache.org). It can be forked, edited and
reused for building packages of NetBeans bundled with a JDK runtime of choice.

## Build

Apache NetBeans releases on GitHub ship source only, so NetBeans is built from
source as part of the package build. Links and hashes of the NetBeans source
release, the bundled JDK runtimes, and the
[NBPackage](https://github.com/apache/netbeans-nbpackage/) tool are configured
in the `build.properties` file. This file also contains the version number to
write into each package, and the NetBeans cluster configuration to build. Other
configuration for individual packages is included in the `config` files passed
in to NBPackage.

Packages are built using the workflow at `.github/workflows/build.yaml`. This
workflow is triggered manually from the `Actions/Build` page using the
`Run workflow` button and selecting the required packages.

NetBeans itself is platform independent, so it is built once by the
`Build NetBeans from source` job and shared with each of the package jobs as a
workflow artefact. The build takes considerably longer than packaging, so its
output is cached against the contents of `Exec.java` and `build.properties` -
repeat runs against unchanged configuration reuse the cached distribution
instead of building again.

The workflow uses the `Exec.java` single source file to download and verify the
required files, to build NetBeans from source with
[Apache Ant](https://ant.apache.org/), and to configure and execute NBPackage.
All downloads are verified against the hashes provided in the `build.properties`
file.

### Building locally

Building requires JDK 21 or above, and Ant on the path (or `ANT_HOME` /
`ANT_PATH` set). To validate the configuration, build NetBeans from source, and
then build a package -

```
java Exec.java validate
java Exec.java build-netbeans
java Exec.java build <os> <arch> <package-type>
```

The source release is extracted to `nbsrc` and built there. The resulting
NetBeans distribution is written to `nbdist`, from where `build` picks it up.
Delete `nbsrc` to force a clean rebuild - `build-netbeans` reuses the directory
if it already exists.

Note that the NetBeans sources contain paths of up to 245 characters, so
extracting them on Windows requires either long path support to be enabled or a
short working directory. The workflow builds NetBeans on Linux.

### Updating the NetBeans version

Set `netbeans.version` to the version to write into the packages, and point
`netbeans.source.url` at the source archive of the matching tag on the
[Apache NetBeans releases page](https://github.com/apache/netbeans/releases).
The hash for `netbeans.source.sha` can be generated with -

```
curl -L <url> | sha512sum
```

Note that the version is also used as the build number of the NetBeans build,
as the source archive does not include the VCS metadata the NetBeans build
otherwise derives it from.

If a release tag is provided when triggering the workflow, installers will be
uploaded to this release. If no release tag is provided, the installers will be
uploaded as workflow artefacts, and can be found in zip files on the workflow
output page. Any release tag must correspond to an already created release,
although this may be in draft form.

## Code signing and secrets

The workflow will optionally sign Windows and macOS installers during the build
if the relevant repository secrets and variables have been defined. Signing is
triggered by the presence of certain variables (marked) - all related secrets
must then be defined.

### Variables

- `APP_CERT_ID` : the name of the Apple Developer Application Certificate passed
  in to `codesign` during the macOS installer build. Presence of this variable
  triggers signing, and all other related macOS variables and secrets must be
  added.
- `AZURE_CERT_PROFILE_NAME` : the name of the certificate profile used by the
  Azure Artifact Signing action during the Windows installer build. Presence of
  this variable triggers signing, and all other related Windows secrets must be
  added.
- `INST_CERT_ID` : the name of the Apple Developer Installer Certificate passed
  in to `pkgbuild` during the macOS installer build.

### Secrets

- `APP_CERT_BASE64` : the Apple Developer Application Certificate exported as
  base64 (see related [GitHub documentation][github-macos]). Required for macOS
  installer signing.
- `APP_CERT_PASSWORD` : password for the Apple Developer Application Certificate.
  Required for macOS installer signing.
- `AZURE_CLIENT_ID` : the `azure-client-id` parameter for the Azure Artifact
  Signing action. Required for Windows installer signing.
- `AZURE_CLIENT_SECRET` : the `azure-client-secret` parameter for the Azure
  Artifact Signing action. Required for Windows installer signing.
- `AZURE_CODE_SIGNING_NAME` : the `signing-account-name` parameter for the Azure
  Artifact Signing action. Required for Windows installer signing.
- `AZURE_ENDPOINT` : the `endpoint` parameter for the Azure Artifact Signing
  action. Required for Windows installer signing.
- `AZURE_TENANT_ID` : the `azure-tenant-id` parameter for the Azure Artifact
  Signing action. Required for Windows installer signing.
- `INST_CERT_BASE64` : the Apple Developer Installer Certificate exported as
  base64. Required for macOS installer signing.
- `INST_CERT_PASSWORD` : password for the Apple Developer Installer Certificate.
  Required for macOS installer signing.

## Current build limitations

The macOS packages will be code signed in the build if the relevant secrets and
ID variables are added to the GitHub repository. For public distribution the
files must be downloaded, notarized and stapled. Optional support for doing this
in the workflow will be added at a later date.

## Legal

These packages are provided without warranty, and under the licenses and terms of
the bundled software. Please make sure you're familiar with all terms before
downloading any artefacts. Apache NetBeans is provided under the terms of the
[Apache Software License](https://github.com/apache/netbeans/blob/master/LICENSE).

Apache, Apache NetBeans and the Apache NetBeans logo are trademarks or registered
trademarks of the Apache Software Foundation. Java and OpenJDK are registered
trademarks of Oracle and/or its affiliates. All other trademarks are the property
of their respective holders and used here only for identification purposes.

[github-macos]: https://docs.github.com/en/actions/how-tos/deploy/deploy-to-third-party-platforms/sign-xcode-applications
