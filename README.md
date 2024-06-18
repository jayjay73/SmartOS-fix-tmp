# Fix /tmp in SmartOS zones

If you run SmartOS Zones on a SmartOS hypervisor the /tmp directory inside the yones will be mounted on tmpfs.

While this is OK in most use cases you will run into problems if you need filesystem features that Solaris/Illumos tmpfs does not provide, like ACLs. One such case is making sshd's forwarded agent socket available to other users. Unlike ssh-agent, which can be persuaded to use a different location for it's socket (-a option) this is not possible with sshd (the consuming side of an agent connection).

You can't just remove /tmp from /etc/vfstab either...

A SMF service instance (svc:/smartdc/mdata:fetch) re-creates the vfstab entry every time on boot if it doesn't find one. In my opinion a very misleading use of a service called mdata-fetch.

## The Fix

1. Copy the manifest and startup script
    ```
    cp mdata-fetch-NO-TMP /opt/local/lib/svc/method/mdata-fetch-NO-TMP
    cp mdata-NO-TMP.xml /opt/local/lib/svc/manifest/mdata-NO-TMP.xml
    ```
1. Stop, disable and then delete the mdata service
    ```
    svcadm stop svc:/smartdc/mdata
    svcadm disable svc:/smartdc/mdata
    svccfg delete svc:/smartdc/mdata
    ```
1. Import the new mdata service
    ```
    svccfg import /opt/local/lib/svc/manifest/mdata-NO-TMP.xml
    ```
1. (Force-) Unmount /tmp
    ```
    umount /tmp
    umount -f /tmp
    ```
1. Remove /tmp from vfstab
    ```
    vim /etc/vfstab # remove line for /tmp or comment it out
    ```
1. Remove the mountpoint /tmp and link /var/tmp there
    ```
    rmdir /tmp
    ln -s /var/tmp /tmp
    ```
1. Enable the new mdata service
    ```
    svcadm enable svc:/smartdc/mdata
    ```

