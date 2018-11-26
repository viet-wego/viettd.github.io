Title: Create & Format a Disk on GCE
Date: 2018-11-08 18:40
Modified: 2018-11-08 18:59
Category: devops
Tags: devops, gcloud, linux, gcp, gce, vm, disk
Slug: create-n-format-a-disk-on-gce
Authors: Viet Tran
Summary: How to create, format and use a disk on Google Compute Engine.

This is the short version to help you do the task. If you want to learn more please check the References at the end of this article.

## Create a new disk

```bash
gcloud compute disks create dev-postgres-master --size=500GB --type=pd-ssd
```

- `dev-postgres-master` is my disk name
- `type` can be `pd-ssd` or `pd-standard`

## Create a temporary instance & attach the disk

```bash
gcloud compute instances create viet-temp
gcloud compute instances attach-disk viet-temp --disk=dev-postgres-master
```

- `viet-temp` is my instance name
- `dev-postgres-master` is name of the disk created above

## Format the new disk

SSH to the instance has the new disk

```bash
gcloud compute ssh viet-temp
```

When you're inside the instance:

- List all devices are attached to server

```bash
sudo lsblk
```

You will get the output like this

```bash
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sdb      8:16   0  500G  0 disk
sda      8:0    0   64G  0 disk
└─sda1   8:1    0   64G  0 part /
```

- In this case, my new disk is `/dev/sbd`, I will format it with these commands

```bash
sudo mkfs.ext4 -m 0 -F -E lazy_itable_init=0,lazy_journal_init=0,discard /dev/sdb
sudo mkdir -p /data
sudo mount /dev/sdb /data
sudo chmod -R 777 /data
sudo umount /data
```

Then exit the server.

- Detach the disk & delete the temporary instance

```bash
gcloud compute instances detach-disk viet-temp --disk=dev-postgres-master
gcloud compute instance delete viet-temp
```

Now, your new disk is ready to use as a `ext4` volumne.

---

## References

- [Adding or Resizing Persistent Disks](https://cloud.google.com/compute/docs/disks/add-persistent-disk)

- [gcloud compute disks create](https://cloud.google.com/sdk/gcloud/reference/compute/disks/create)