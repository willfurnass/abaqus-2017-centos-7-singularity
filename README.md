# Singularity container for Abaqus 2017

Abaqus works well under Centos but is much less likely to work well under a bleeding edge distro like Arch, so let's run Abaqus within a Singularity container.

## Building the container

We unpack the installer ISO during the build then kick off the Abaqus installer, using 'UserIntentions' files to automate the install.


```sh
# Set the Singularity cache dir to somewhere with lots of space; 
# by default Singularity uses /tmp for temp files, which is 'only' 16GB on my laptop (too small in this case)
mkdir -p m 0700 $HOME/.cache/singularity 

sudo SINGULARITY_CACHEDIR=$HOME/.cache/singularity singularity build test.img centos-7-abaqus-2017-singuarity-notes.def
```

## Running the container

sudo singularity shell --bind some/dir:/iso-unpacked another/dir:/cfgs centos-7-abaqus-2017-singuarity-sandbox

## How the 'UserIntentions' files were generated

These were produced by going through the Abaqus install process interactively within a Singularity 'sandbox' (writable directory-based container):

```sh
pushd some/dir
tar xfp Abaqus2017.iso
popd

# Build a writable container in a directory called 'centos-7-abaqus-2017-singuarity-sandbox'
sudo singularity build --sandbox centos-7-abaqus-2017-singuarity-sandbox  docker://centos:7

# Get a shell in that container
sudo singularity shell --bind some/dir:/iso-unpacked another/dir:/cfgs centos-7-abaqus-2017-singuarity-sandbox

# Within the container...
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
yum install -y ksh redhat-lsb-core perl && yum clean all
cd /iso-unpacked
# Run the Abaqus installer
ksh 1/StartTUI.sh
# Find the '*UserIntentions* files generated during the install process and save them outside the container
```
