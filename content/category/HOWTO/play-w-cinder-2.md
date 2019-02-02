Title: Let’s play with Cinder and RBD (part 2) 
Date: 2016-07-29 10:02

![OpenStack logo]({filename}/images/openstack_slide.png)
The approach is to encourage you to get familiar with Cinder.
Show what Cinder can do and how RBD performs.

1. Create a volume
2. Create a snapshot
3. Create a volume from a snapshot
4. Create a volume from another volume


# Create volume
First, we need to create a logical volume:

    :::python
    $ cinder create 1 --name v1 


Positional arguments:

- <size> : Size of volume, in GiBs. (Required unless snapshot-id/source-volid is specified).
- Check the help command: `cinder help create` for more info.

Let’s verify with ~/sudo rbd ls volumes to check what rbd have: 

    :::python
    $ sudo rbd ls volumes
    >bar
    >volume-1b681f9f-81f6-4965-ad89-28ffb10c1ede

`bar` is an image created with RBD client, you can see that all
the "Cinder volumes" begins with `volume-<uuid>.`
Be aware that RBD and Ceph call *image* what Cinder identify as *volume.*
                                                                   
# Create snapshot
Therefore in RBD, everything is thinly provisioned, snapshots and clones
use **copy-on-write**.

When you have a [copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write#Copy-on-write_in_storage_media)
(cow) snapshot, it means it has a dependency
on the parent. The single blocks in the snapshot will be the blocks that
have been modified (and thus copied) that makes snapshot creation very
very fast (you only need to update metadata), and data doesn’t actually
move or copy anywhere  …. instead, it’s copied on demand, or on write!

This dependency means that you cannot, e.g., delete a volume that has
snapshots because that would make those snapshots unusable, like pulling
the rug out from under them.

    :::python
    $ cinder snapshot-create <volume ID or name> --name snap1
    $ cinder snapshot-create volume1 --name snap1
    
    $ sudo rbd ls -l volumes
    >NAME SIZE PARENT FMT PROT LOCK 
    >bar 1024M 1 
    >volume-1b681f9f-81f6-4965-ad89-28ffb10c1ede 1024M 2 
    >volume-1b681f9f-81f6-4965-ad89-28ffb10c1ede@snapshot-6e9..d 1024M 2 yes 

Snaps end with `volume.<cinder volume ID>@snapshot-<ID snap>.`

# Create volume from snapshot

From: `cinder snapshot-list` get the snap ID.

    :::python
    $ cinder create --snapshot-id 6e93e928-2558-4f12-a9ab-12d25cd72dbd --name v-from-s

# Create a volume from another volume

Since we are cloning from a volume and not a snapshot, we must first create a
snapshot of the source volume.

    :::python
    $ cinder create --source-volid 1b681f9f-81f6-4965-ad89-28ffb10c1ede --name v-from-v
    $ sudo rbd ls -l volumes

In this case, the clone volume has a `volume-ID@volume-ID.clone_snap.`

# READ MORE

+ [Ceph docs: how to create a volume](http://docs.ceph.com/docs/hammer/rbd/rados-rbd-cmds/)
+ [Ceph docs: OpenStack](http://docs.ceph.com/docs/master/rbd/rbd-openstack/)
+ [Ceph docs: Snapshots](http://docs.ceph.com/docs/master/rbd/rbd-snapshot/)

The Logs are always relevant. Check the log files used by Block Storage:
`sudo journalctl -f -a --unit devstack@c-bak.service.` 
