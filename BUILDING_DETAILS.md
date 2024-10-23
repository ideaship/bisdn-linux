# Building details

This document covers details that may be useful to some users.

## Building without containers

We recommend building with containers to ensure a reproducible
build. Building without containers has been known to fail on recent
Linux distributions due to incompatible versions of build tools.

If you want to build without using containers, you can replace the
`kas-container` command with `kas` in a suitable build environment.

On Ubuntu 22.04 LTS (jammy), you would install the required packages
and then run kas:

```shell
sudo apt-get install --no-install-recommends \
  build-essential chrpath diffstat lz4
kas build bisdn-linux.yaml
```

## Building under disk space constraints

If your build runs out of disk space, you will see an error like this:

```
ERROR: No new tasks can be executed since the disk space monitor action is "STOPTASKS"!
ERROR: Immediately halt since the disk space monitor action is "HALT"!
```

To work around this, you can include the [`rm-work.yaml`](rm-work.yaml)
file in your kas configuration. This will remove intermediate files
after a package has been successfully built which reduces the amount
of disk space required substantially:

```shell
kas-container build bisdn-linux.yaml:rm-work.yaml
```

## Limiting memory and CPU usage

Even if you have the recommended RAM and CPU cores, the build process may
expand to eat all resources on your build host and make it unresponsive
to the point where only resetting the machine will help. To avoid this,
you can limit the resources available to the build process by having
kas-container pass additional options to `docker run`. For instance:

```shell
kas-container --runtime-args "--cpus=14" --runtime-args "--memory=16g" \
  --runtime-args "--memory-swap=20g" build bisdn-linux.yaml:ofdpa-gitlab.yaml
```

## Building ofdpa-gitlab

If you want to build ofdpa from source, you need access to the
ofdpa-gitlab repo. The most convenient way to authenticate when building
with `kas-container` is to use your own `.ssh` directory (to avoid
complaints that the authenticity of the host can't be established)
and your ssh-agent (for authentication with gitlab.bisdn.de):

```shell
kas-container --ssh-agent --ssh-dir ~/.ssh build bisdn-linux.yaml:ofdpa-gitlab.yaml
```

## Troubleshooting

If your build fails, it may be because your [`build`](build) directory
is in an inconsistent state (this can happen, for instance, if the build
process is killed). The `build` directory is rebuilt if you remove it
(or parts of it). Rebuilding `build/downloads` and `build/sstate-cache`
can take a long time, so you may want to remove just `build/tmp` and
restart the build. If that works, there is no need to remove all of
`build`.