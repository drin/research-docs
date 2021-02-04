# Experimental Setup

## Data Locations

The top-level directory, which is also the mountpoint for the ZFS pool, is `experimentdata`:
`/experimentdata`. Later in this documentation, we refer to this directory location as
`${zfs-datadir}`.


# Section 2
I also created the user disorderly_guest, and you should be able to connect using:

psql -hlocalhost xhca

The database name is xhca, and I granted all privileges to disorderly_guest:

GRANT ALL ON DATABASE xhca TO disorderly_guest;

The owner is my user (akmontan) and you can see in the experimentdb script that I switch to the "postgres" user to issue DB admin commands (pg_ctl <command>).

The cloudlab server has ~120G of RAM and 32 processors. If you aren't totally comfortable with
linux, here are some convenient commands you can check out:
* free
* nproc
* lsblk (I typically use -a)
* zpool (e.g. `zpool list` and `zpool status <pool name>`
* zfs
* gdisk
* ls /dev/disk (this directory has symlinks to each device using a variety of symlink names, such as UUID, filesystem label, etc.)


# Section 4
Cloudlab servers are leased for a default of 16 hours, and typically you don't want to extend them
much more than a week.

Whenever you want to try playing around with cloudlab, you can signup and request to join project
"Skyhook". Once you get through all of that, you should be able to access a project profile I setup
called "psql-test". A profile is basically a hardware configuration.

We tend to use Ubuntu 18.04 (I think) for our images, and so cloudlab servers are booted as a fresh
install of that image. From there, everything has to be setup (e.g. postgres server, formatting
hard drives, etc.). I have some installers I wrote for my own convenience, which can be found in my
config repo (cloudlab branch). Specifically, here's a small script to install postgres:
https://github.com/drin/configs/blob/cloudlab/installers/install-postgres.ubuntu

optionally, here are some other scripts I use for my own environment setup:
https://github.com/drin/configs/blob/cloudlab/installers/install-tools.bash
https://github.com/drin/configs/blob/cloudlab/installers/setup-environment.bash
https://github.com/drin/configs/blob/cloudlab/installers/install-vim-plugins.bash

it's easiest to clone the whole repository, set CONFIG_ROOT to where you cloned the repository, and then run any of those scripts with `bash <script>` (except the vim plugins one, which must be run as `bash install-vim-plugins.bash setup`). You don't have to use these scripts, but feel free to use them for reference.

I think otherwise, you can reference my previous emails for how I setup the cloudlab server before (paths and stuff) and try to set it up similarly.

If you want to try setting up ZFS, you can use this script for reference: https://gitlab.com/xhca/xhcadb/-/blob/mainline/shell-scripts/infrastructure/zfs.create.bash.


<!-- resources -->
