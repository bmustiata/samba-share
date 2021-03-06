

Samba Docker volume sharing plugin
==================================

Sharing a Docker container's volume should be as simple as `docker run bmst/samba-share <container> | sh`.

This 'plugin' will create and configure a samba server container that auto-creates shares for all
the volumes attached to the specified container, and also all the mounts that are made to it.

Usage
-----

Possible scenarios are

- `docker run bmst/samba-share <container> | sh` shares the volumes of `<container>`.
- `docker run bmst/samba-share` reminds the user what the options are.
- Additional parameters can be [passed as environment variable](https://docs.docker.com/engine/reference/run/#env-environment-variables) and can be combined. Possible names are USER PASSWORD USERID GROUP READONLY RUN_ARGUMENTS STATIC_IP. Examples:

        # run a samba server in read only mode
        docker run -e READONLY=yes bmst/samba-share <container> | sh
        # run a samba server on the host and share the content to other computers
        docker run -e RUN_ARGUMENTS="--net=host" bmst/samba-share <container> | sh

    - USER is the samba user (default: "root")
    - PASSWORD is USER's password (default: "tcuser")
    - USERID to use for the samba USER (default: "1000")
    - GROUP user group (default: "root")
    - READONLY "yes" or "no" whether write access is denied (default: "no")
    - RUN_ARGUMENTS which additional arguments to pass to the `docker run ... samba-server` (default: "")
    - STATIC_IP what IP on the host to bind the service to. (default: "")

    Warning: If you use a `\` in these variables, it could be removed or unescaped.

Try it out
----------

Create a volume in my-data and share its content via samba

    # Make a volume container (only need to do this once)
    docker run -v /data --name my-data busybox true
    # Share it using Samba (Windows file sharing)
    docker run bmst/samba-share my-data | sh

How it works
------------

The `bmst/samba-share` container uses the bind-mounted docker client and socket to introspect
the configuration of the specified container and, then uses that information to setup a new container
that is `--volumes-from` setup to give it access.

Tested
------

-
        Client:
          Version:      1.9.1
          API version:  1.21
          Go version:   go1.4.2
          Git commit:   a34a1d5
          Built:        Fri Nov 20 13:20:08 UTC 2015
          OS/Arch:      linux/amd64

        Server:
          Version:      1.9.1
          API version:  1.21
          Go version:   go1.4.2
          Git commit:   a34a1d5
          Built:        Fri Nov 20 13:20:08 UTC 2015
          OS/Arch:      linux/amd64

Credits
-------

This was derived from [`svendowideit/samba`](https://github.com/SvenDowideit/dockerfiles/tree/master/samba) to [`niccokunzmann/samba-share`](https://github.com/niccokunzmann/dockerfiles/tree/master/samba-share) because of [Issue 29](https://github.com/SvenDowideit/dockerfiles/issues/29).
This was derived from [`niccoknzmann/samba-share`](https://github.com/niccokunzmann/dockerfiles/tree/master/samba-share) to [`bmst/samba-share`](https://github.com/niccokunzmann/dockerfiles/tree/master/samba-share) because it wasn't working, and was including a bunch of unnecessary other docker images.

Wasn't working means:

* Unable to create multiple servers for different hosts.
* Didn't exported mounted folders
* Unable to assign static IPs to the server.
