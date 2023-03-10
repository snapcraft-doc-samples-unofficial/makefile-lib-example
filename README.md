# Set up classic confinement for a Makefile project

Some snaps need to have access to system resources outside the scope allowed by
strict confinement, and are unable to do this via the available interfaces.
These snaps are configured to use classic confinement.

This guide shows how to enable classic confinement for a snap built with the
`make` plugin. The example project used in this guide can be found in [this repository](https://github.com/snapcraft-doc-samples-unofficial/makefile-lib-example).

## Change the confinement to classic

Starting with an existing `snapcraft.yaml` file, change the `confinement` setting to `classic`:

```yaml
confinement: classic
```

This will cause the snap to be built in a way that gives it access to system resources.

## Use linters to identify problems

Snapcraft uses [linters](https://snapcraft.io/docs/linters) to check for issues during builds.
Linters can only be specified in snaps that use the [core22](https://snapcraft.io/docs/base-snaps) base. Warnings are still reported for snaps that use older bases.

Run snapcraft to build the snap. This may produce warnings like the following:

```
Lint warnings:
- classic: bin/example-make-lib: ELF interpreter should be set to '/snap/core22/current/lib64/ld-linux-x86-64.so.2'.
- classic: bin/example-make-lib: ELF rpath should be set to '/snap/core22/current/lib/x86_64-linux-gnu'.
```

If there are many warnings about libraries you can disable the library linter so that only classic linter warnings are shown. See the [linters](https://snapcraft.io/docs/linters) documentation for details.

## Fix linter warnings in the Makefile

In this example, the warnings about the ELF interpreter and rpath can be handled by adding options to the linker in the appropriate build rule of the Makefile:

* `-Wl,-dynamic-linker=/snap/core22/current/lib64/ld-linux-x86-64.so.2`
* `-Wl,-rpath=/snap/core22/current/lib/x86_64-linux-gnu`

If the `LDFLAGS` environment variable is used in the Makefile, the `snapcraft.yaml` file could be updated to pass these options to the `make` plugin, like this:

```yaml
    plugin: make
    make-parameters:
      - LDFLAGS="-Wl,-dynamic-linker=/snap/core22/current/lib64/ld-linux-x86-64.so.2 -Wl,-rpath=/snap/core22/current/lib/x86_64-linux-gnu"
```

This will only be useful for projects where the `snapcraft.yaml` file is maintained as part of the build system.

## Rebuild the snap

Run Snapcraft again to rebuild the snap, consulting the [Classic linter](https://snapcraft.io/docs/linters-classic) documentation to resolve further issues.

See also [this article ](https://snapcraft.io/blog/the-new-classic-confinement-in-snaps-even-the-classics-need-a-change) for an overview of the classic linter and a discussion of the issues involved in building snaps for classic confinement.
