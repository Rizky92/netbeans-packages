# Apache NetBeans package builds

This repository contains a configuration and build system for packages of
[Apache NetBeans](https://netbeans.apache.org). It can be forked, edited and
reused for building packages of NetBeans bundled with a JDK runtime of choice.

## Build

Links and hashes of Apache NetBeans, the bundled JDK runtimes, and the
[NBPackage](https://github.com/apache/netbeans-nbpackage/) tool are configured
in the `build.properties` file. This file also contains the version number to
write into each package. Other configuration for individual packages is
included in the `config` files passed in to NBPackage.

Packages are built using the workflow at `.github/workflows/build.yaml`. This
workflow is triggered manually from the `Actions/Build` page using the
`Run workflow` button and selecting the required packages.

The workflow uses the `Exec.java` single source file to download and verify the
required files, as well as to configure and execute NBPackage. All downloads are
verified against the hashes provided in the `build.properties` file.

At the end of the workflow run, each built installer can be found in a zip file
on the workflow output page.

## Current build limitations

The Windows installer must currently be downloaded and code signed locally.
Optional support for signing in the workflow will be added at a later date.

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
