# podman

## ERRO[0000] cannot find UID/GID for user

If you see the following error:

```
ERRO[0000] cannot find UID/GID for user pat: no subuid ranges found for user "MYUSER" in /etc/subuid - check rootless mode in man pages.
```

This means your current user does not allow creating rootless containers (containers that run as a non-root user), because a requirement is missing: subuid and subguid ranges for the user.

Most recent operating systems will do this for you, but if your user was created on an old version of such operating system, and then you kept upgrading, it is possible subuid and subguid are missing.

How to fix this problem is [documented](https://github.com/containers/podman/blob/main/docs/tutorials/rootless_tutorial.md#etcsubuid-and-etcsubgid-configuration)

Assuming there are no ranges assigned (check `/etc/subuid` and `/etc/subguid`), you can fix the issue by running the following command (replace `MYUSER` with your username):

```
sudo usermod --add-subuids 100000-165535 --add-subgids 100000-165535 MYUSER
```

## Wrong filesystem error

If you see the following error:

```
Error: ".local/share/containers/storage/btrfs" is not on a btrfs filesystem: prerequisites for driver not satisfied (wrong filesystem?)
```

This issue appeared when using the XFS filesystem for my `/home` directory. By default rootless containers are configured to use a supported driver for the root file system. To ensure general compatibility set the container driver to `overlay`. For more information see: `man 5 containers-storage.conf`.

 As root modify `/etc/containers/storage.conf`: 

```
[storage]

# Default Storage Driver, Must be set for proper operation.
#driver = "btrfs"
driver = "overlay"
```

Next resolve the issue by wiping the entire `/var/lib/containers` directory.

**CAUTION! This will delete all volumes in the directory! Do not do this without backing up the directory, copying your containers elsewhere, or making sure you don't need anything that exists there! Before executing this action stop all running containers with: `podman-stop --all`**

```
rm -rf /var/lib/containers
```

You should now be able to build documentation correctly using podman.
