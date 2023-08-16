# Setting up a GCP VM for JupyterLab

The simplest way to get up and running on GCP is to use the [Deep Learning VM image](https://cloud.google.com/deep-learning-vm) from Google. It will automatically start jupyterlab for you, and you can mount a big data disk at `/home/jupyter`.

But why would you do that when you can follow this guide and do almost the same thing with a little more customization?

The main advantages of doing it this way:

 * We can install `mamba` from the beginning
 * We can use the latest Python and `jupyterlab` versions
 * Easier access to everything from within the `jupyterlab` terminal itself. The DL VMs make it difficult to install things

None of the stuff on this page is unique to the MDL, so I hope it's useful for other folks too.

## Starting from scratch

Step 1 is to create a fresh VM. I've been using whatever the latest Debian version is.

For now, we don't need a lot of CPU power or memory, so a standard 2-vCPU instance is fine. It's easy to shut down an instance, change the size, and restart it later.

### Disk configuration

GCP charges for disks based on the type and the size. The default is a `balanced persistent disk` which offers faster access but at increased cost. For large data disks, it is better to use a `standard persistent disk`. It's easy to make a disk bigger on the fly if you need more space (see [below](#expanding-the-size-of-a-disk)), but it's a pain to migrate to a smaller disk. So there are two options for setting up a VM:

 * Use one big disk for the OS, your code, packages, and the data
 * Create a fast boot disk for the OS and code, and attach a separate data disk that's big
 
The benefit of using one disk is that it makes life simpler. The downside is that you have a big disk sitting around with all your stuff on it. Eventually your environment might become too messy but you won't want to touch anything because all the data is there.
  
Another concern is that eventually the VM image you used might become _deprecated_ and GCP won't let you use it (at least, this seemed to happen to me sometimes). At that point, you'll want to create a new VM and use the old disk as a data drive.

The benefit of using two disks is that you can keep the software on a faster disk, and only use the standard persistent disk for data. When you've finished with a particular analysis, you can copy it all up to a bucket and remove the data disk. Then you can create a fresh one for a new dataset.

Alternatively, you could detach the data disk and attach it to a new VM with updated software. It's nice to be able to create a fresh VM on the fly with new dependencies, when desired.

#### Expanding the size of a disk

If you end up needing more space, you can expand a disk even while the VM is running. [Google's documentation](https://cloud.google.com/compute/docs/disks/resize-persistent-disk) explains this step-by-step. For a data disk, it's as simple as resizing and then running `resize2fs` on the device.

**Note:** You can't _shrink_ a disk this way. Try to be conscientious about creating huge disks, as we have to pay for that storage. If you need a lot of disk space temporarily, you can create a big disk and then delete it when you're done (saving final outputs in a bucket).

## Setting up the VM

With either configuration, the first step is to start the instance and `ssh` into it, using your own credentials. You're going to configure things from here, but in the future you can do everything from the `jupyter` user.

### TL;DR version:

You can download the necessary scripts from [our Github](https://github.com/MethodsDev/gcp-vm/releases) and run `sudo bash setup.sh`:

```bash
# wget from github and extract to /tmp
wget https://github.com/MethodsDev/gcp-vm/releases/download/v0.1/gcp-vm.tgz -O - | tar -xz -C /tmp

# (if needed) figure out the name of the data disk by running:
lsblk
# the output will list the attached disks.
# your data disk is the large one that isn't attached to anything
# it's *probably* going to be /dev/sdb but sometimes it isn't

sudo bash /tmp/gcp-vm/setup.sh [/dev/sdb]  # leave the argument blank if not needed
```

And then restart the instance with `gcloud compute instances reset [instance-name]`. Once it has restarted, you should be able to connect to `jupyterlab`.

### Step by step version

Here I'll go through the `setup.sh` script and explain how it works, for posterity.

If we are using a separate data disk, we need to format it.

```bash
DEV=$1
if [[ -z ${DEV} ]]; then
	echo "No device argument provided, assuming this is a single-disk instance"
elif [[ -n ${DEV} ]]; then
	mkfs.ext4 -m 0 -E lazy_itable_init=0,lazy_journal_init=0,discard ${DEV}
fi
```

We'll update packages and install some useful stuff.

```bash
apt update
# curl and git are always good, the others are just tools I use a lot
apt install -y zsh curl git less nano htop
```

Next we create a `jupyter` user and add them to the `google-sudoers` group so that we can use `sudo` from the jupyter terminal.

```bash
# feel free to pick whatever shell you like
adduser --quiet --shell /bin/zsh --disabled-password --no-create-home --gecos "" jupyter
usermod -a -G google-sudoers jupyter
mkdir /home/jupyter
# mount the data disk, if we have one
if [[ -n ${DEV} ]]; then
	mount -o discard,defaults ${DEV} /home/jupyter/
fi
# users should have a .local directory
mkdir /home/jupyter/.local
# make sure jupyter owns its own home directory
chown -R jupyter:jupyter /home/jupyter
```

Now it's time to install `mamba`! This is a faster alternative to `conda`. Because we're customizing things we can install whatever we want, including the latest version of `jupyterlab`. We also need to set up a config file for jupyter. We'll stick this stuff in `/opt` so it lives on the boot disk (if we have two) and will persist for different data disks.


```bash
# downloading the installation script
curl -s -L -o /tmp/mambaforge.sh \
    https://github.com/conda-forge/miniforge/releases/latest/download/Mambaforge-Linux-x86_64.sh

# tell it to install to `/opt/mamba` so it lives on the boot disk
# specifying HOME so it is installed for the jupyter user
HOME=/home/jupyter bash /tmp/mambaforge.sh -s -b -p /opt/mamba

# install jupyterlab
/opt/mamba/bin/mamba install -y jupyterlab
mkdir /opt/jupyter
mv /tmp/jupyter_notebook_config.py /opt/jupyter/
# we need the jupyter user to own these so we can install stuff
chown -R jupyter:jupyter /opt/mamba /opt/jupyter
```

### Setting up automated startup

The final trick is to make our VM start up all these things on boot. When we start the VM, we want it to mount the data disk (if present) and start `jupyter lab` running with the configuration we specified. These scripts accomplish that.

```bash
mv /tmp/gcp-vm/etc/env.sh /etc/profile.d/
chown root:root /etc/profile.d/env.sh

mkdir /opt/bin
mv /tmp/gcp-vm//bin/{define-jupyter-service,mount_workspace}.sh /opt/bin/
chown root:root /opt/bin/{define-jupyter-service,mount_workspace}.sh
```

The `rc.local` script will run on boot and calls the other scripts. It needs to know how to mount our data disk. It's possible for the device name to change on a reboot, so we're going to get the UUID for the device and stick it in this script.

```bash
# going to create this script on the fly to insert device info
UUID=`lsblk -n -o UUID ${DEV}`

cat <<-EOH > /etc/rc.local
#!/bin/bash
/opt/bin/mount_workspace.sh /home/jupyter ${UUID}
/opt/bin/define-jupyter-service.sh
EOH

chown root:root /etc/rc.local 
chmod +x /etc/rc.local 
```

Finally we enable the daemon to run everything on startup.

```bash
systemctl daemon-reload
systemctl start rc-local
```

At this point, we can exit our ssh session and `gcloud compute instances reset` the instance. When it finishes booting up, we should be able to use `gcloud compute ssh -- -NL 8080:localhost:8080` to connect to jupyterlab.

#### Acknowledgements

A couple of the scripts here were adapted from the stuff Google installs on their VMs. So I'll thank them for making it all available, and mildly chide them for making it this difficult to customize everything. 🙂
