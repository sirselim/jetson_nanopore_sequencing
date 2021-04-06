|:exclamation: please be fore warned that by following this set up guide neither the authors or Oxford Nanopore Technology (ONT) are liable for any hardware/consumable damage or data loss.|
|-------------------|

# Setting up GPU live basecalling

Before beginning it is recommended that you have registered for the ONT community portal and have access there. It is a great and helpful place to go to ask questions of the Nanopore community and ONT experts. If you haven't joined the community you can do so [here](https://community.nanoporetech.com/). If you are part of an institute that have an account with ONT it is likely that you can be added under that, so it's worth asking around to see if someone can do that for you.
## Preamble

What I am going to detail below is my variation on the instructions that were created by ONT ([here](https://community.nanoporetech.com/posts/enabling-gpu-basecalling-f)). The instructions at this link, as of the **6<sup>th</sup> April 2021** are slightly out of date and don't reflect the current complete set up to get up and running with GPU live basecalling. So what follows is my interpretation of the instructions. I have stated above and I will reiterate here, ***by following the below you recognise that neither myself or Oxford Nanopore Technologies are responsible for any damage to hardware, software, or consumables as well as data*** - that's all on you. So if you aren't comfortable 'tinkering' around in Linux then I suggest seeking additional support.

Below there may well be dragons, you have been warned enough, so without further ado lets move forward shall we?
## Caveats

The below caveats have been sourced from this ONT community forum post ([link](https://community.nanoporetech.com/posts/enabling-gpu-basecalling-f)). 

* **GPU basecalling is supported on Linux and NVIDIA GPUs only.**
  * The GPU must be installed on the same system to which the MinION is connected.
    * This is done at the user's own risk: mis-configuration of the GPU may result in slow basecalling and/or a large number of skipped reads.
      * i.e. if the basecall server crashes as a result of mis-parameterization. 
* GPUs with a low amount of memory (**less than 4 GB**) may not work at all.

## The process

OK, so the below steps go into detail around how to get up and running. They also include a little discussion/insight around some of the 'pitfalls' and particular quirks that may arise.
### Ensure you have MinKNOW installed

I'm making some assumptions when writing this, and one of these is that you are comfortable working in a Linux environment. Another is that you probably already have a Linux computer set up with ONT software (i.e. the correct repositories). The below is the code to do this but I won't delve any deeper than this as this particular document is concerned with getting live GPU basecalling up and running.

Another assumption is that you are running Ubuntu 18.04, if not I'm assuming that you know what you are doing!

##### Update system and add ONT repo (Ubuntu 18.04)

```sh
sudo apt-get update
sudo apt-get install wget
wget -O- https://mirror.oxfordnanoportal.com/apt/ont-repo.pub | sudo apt-key add -
echo "deb http://mirror.oxfordnanoportal.com/apt bionic-stable non-free" | sudo tee /etc/apt/sources.list.d/nanoporetech.sources.list
```

##### install MinION software

```sh
sudo apt-get update
sudo apt-get install minion-nc
```

##### check version of software

```sh
apt policy minion-nc
```

You should see something like this as output of the above:

```sh
minion-nc:
  Installed: 21.02.1-1~xenial
  Candidate: 21.02.1-1~xenial
  Version table:
 *** 21.02.1-1~xenial 500
        500 http://mirror.oxfordnanoportal.com/apt xenial-stable/non-free amd64 Packages
        500 http://mirror.oxfordnanoportal.com/apt xenial-stable/non-free i386 Packages
        100 /var/lib/dpkg/status
```

If you are having issues with getting MinION software / MinKNOW setup I suggest you go to the community forum and search for similar questions/issues, and if nothing turns up you can create your own.
### Gettng the correct version of Guppy

This is something that seems to cause constant confusion, and I can see why! The versions of MinKNOW and Guppy are tightly tied together, **BUT** it doesn't mean that the latest version of each piece of software works with each other... with me so far?

What this means in practice is that currently the latest Linux version of MinKNOW for MinION is 21.02.1, which requires Guppy version 4.3.4. However, the latest and greatest version of Guppy is currently 4.5.2. Now many people will be tracking that latest Guppy version as it's nice to have all the new bells and whistles, but it **will not** play nicely with MinKNOW and will crash out as soon as you try to run a sequencing experiment with live basecalling. There are ways to have multiple versions present on a system without causing issues, but more on that later.

For now the below table should help you decided which versions of software you need to grab:

| MinION Release | MinKnow Core version | GUI version | Guppy version working  |
|:--------------:|:--------------------:|-------------|------------------------|
|    20.06.5     |        4.0.5         | 4.0.21      | 4.0.9                  |
|    20.06.17    |        4.0.5         | 4.0.21      | 4.0.11, 4.0.14, 4.0.15 |
|    20.10.3     |        4.1.2         | 4.1.22      | 4.2.2, 4.2.3           |
|    21.02.1     |        4.2.5         | 4.2.8       | 4.3.4                  |

*the above is a snapshot of the latest software versions, if you require a more complete overview you can find that [here](https://github.com/sirselim/jetson_nanopore_sequencing#minknow--guppy-compatibility).

So, as stated above, the current MinION MinKNOW release is 21.02.1, and it needs Guppy 4.3.4
### Download the correct version of Guppy

This is where you are going to require access to the ONT community portal. Various pieces of ONT software are distributed (***freely***) from this portal, but only from this portal, i.e. they cannot be shared by third parties. So once you have access to the portal you can continue.

You should check out the software downloads page [here](https://community.nanoporetech.com/downloads), [this](https://community.nanoporetech.com/posts/enabling-gpu-basecalling-f) post, as well as [this](https://community.nanoporetech.com/posts/no-ampere-compatible-guppy) one. These will guide you in where and how to download the version of GPU Guppy that you will require.

Note: the last link above is a post that deals with issues that arose with the new Nvidia Ampere GPUs. This has meant that versions of Guppy have had to be specially 'patched' for Ampere compatibility, however after Guppy 4.3.4 this should be the last time we will see this and the pre-built versions available should work just fine.

### Extract the version of Guppy you downloaded

Once you have downloaded the correct (GPU) version of Guppy you can extract it:

```sh
tar -xzvf ont-guppy_4.3.4_linux64.tar.gz
```

The above example is for Guppy 4.3.4, but you can substitute for the compressed file of the version you downloaded.

This should result in a folder called `ont-guppy`, which if you run `ls` on should give something like this:

```sh
$ls ont-guppy
bin  data  lib
```

The `bin` directory contains the binaries that we are interested in.

### Copy Guppy to `/opt/ont`

The below code assumes that you are currently in the directory where you extracted Guppy above. You should have the `ont-guppy` folder present at this level. The below will copy this directory and everything in it over to `/opt/ont`. You will likely need `sudo` (super user) access to the machine you are working on for this and many following steps.

|:exclamation: this will overwright any version of Guppy that is present at this same location. Make sure this is what you want to do.|
|-------------------|

```sh
sudo cp -r ont-guppy /opt/ont/
```

### Create sym links to `/usr/bin`

Now we have copied `ont-guppy` and all it's contents this is where what I do differs from the suggested instructions. I create symlinks of the newly copied binaries to `/usr/bin/` essentially making this version of Guppy (and in this case ONTs version of minimap2) the system-wide versions of these tools. Now this might not be for everyone, but I'm comfortable with what I am doing and this works well for me. As I stated above I will explain how I deal with other versions of Guppy later on.

**An important note:** this step has been modified after the release of Guppy 4.2.2, which introduces `guppy_basecall_client` in place of the `guppy_basecaller`. This has a HUGE influence when we get to modifying the `app_conf` file soon.

This code will create sym links of the binaries:

```sh
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecaller /usr/bin/guppy_basecaller
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecall_client /usr/bin/guppy_basecall_client
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecall_server /usr/bin/guppy_basecall_server
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecaller_supervisor /usr/bin/guppy_basecaller_supervisor
sudo ln -s /opt/ont/ont-guppy/bin/guppy_barcoder /usr/bin/guppy_barcoder
sudo ln -s /opt/ont/ont-guppy/bin/guppy_aligner /usr/bin/guppy_aligner
sudo ln -s /opt/ont/ont-guppy/bin/minimap2 /usr/bin/minimap2
```

You should now be able to check to see if they are infact being picked up by running the below:

```sh
# can check versions linked are correct
guppy_basecaller --version
guppy_basecall_server --version
guppy_aligner --version
guppy_barcoder --version
guppy_basecall_client --version
minimap2 --version
```

You should see something like this:

```sh
$ guppy_basecall_client --version
: Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 4.3.4+ecb2805, client-server API version 4.0.0
```

Notice that this is the correct version for MinION MinKNOW 21.02.1. Great!



1. Identify the version of the Guppy basecall server that MinKNOW is using:

     /usr/bin/guppy_basecall_server --version

2. Download the archive version of GPU-enabled Guppy from the Nanopore Community.

     The specific URL to use is:

     https://mirror.oxfordnanoportal.com/software/analysis/ont-guppy_<version>_linux64.tar.gz

     Where <version> is the numeric part (major.minor.patch) obtained from step 1. For example:

     https://mirror.oxfordnanoportal.com/software/analysis/ont-guppy_3.2.10_linux64.tar.gz

3. Extract the archive to a folder, e.g.:

     tar -C /home/myuser/ont-guppy -xf ont-guppy_XXX_linux64.tar.gz

 4. Make sure the archive version of Guppy runs on your machine (see the installation section of the Guppy protocol – you may need to install GPU drivers, for example).

 5. Modify MinKNOW's application config to enable GPU basecalling and set appropriate settings (where "myuser"is the appropriate location for your installation):

     sudo /opt/ont/minknow/bin/config_editor --conf application --filename /opt/ont/minknow/conf/app_conf \

    --set guppy.server_executable="/home/myuser/ont-guppy/bin/guppy_basecall_server" \

    --set guppy.client_executable="/home/myuser/ont-guppy/bin/guppy_basecaller" \

    --set guppy.gpu_calling=1 \

    --set guppy.num_threads=3 \

    --set guppy.ipc_threads=2

Information about Guppy settings can be found in the appropriate section of the Guppy documentation on the community.

6. Stop the MinKNOW service:

      sudo service minknow stop

7. Confirm the guppy_basecall_server process isn't running:

      $ ps -A | grep guppy_basecall_

    If the result of the above command is not blank, manually kill the process:

       sudo killall guppy_basecall_server

8. Start the MinKNOW service:

      sudo service minknow start

9. Confirm that guppy_basecall_server is using the GPU:

      nvidia-smi

10. Monitor your first sequencing run using the GUI to make sure basecalling is working as expected.