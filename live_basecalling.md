|:exclamation: please be fore warned that by following this set up guide neither the authors or Oxford Nanopore Technology (ONT) are liable for any hardware/consumable damage or data loss.|
|-------------------|

-----

_last modified: 29nd June 2021_  
Updated inline with recent MinKNOW release (21.06.0, Guppy 5.0.11).
There have been issues with people running 20.04 or other distro's and `minion-nc`.

-----

# Setting up GPU live basecalling

Before beginning it is recommended that you have registered for the ONT community portal and have access there. It is a great and helpful place to go to ask questions of the Nanopore community and ONT experts. If you haven't joined the community you can do so [here](https://community.nanoporetech.com/). If you are part of an institute that have an account with ONT it is likely that you can be added under that, so it's worth asking around to see if someone can do that for you.

You will need to be a member of the ONT community portal to download particular pieces of software, so please go and set that account up before trying to follow along.

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
* As of Guppy 5.0.X RAM consumption is higher than before, meaning that cards with lower amounts of memory (i.e. ~4Gb) will likely need modified parameters to run correctly.

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

-----

ONT are actively developing toward supporting Ubuntu Focal (20.04). While it hasn't been announced yet you can access the repository. **WARNING:** The below is completely untested by anyone other than myself, so if you aren't comfortable with what you are doing and don't know how to troubleshoot Linux issues I'd advise not to proceed.

The Focal sources can be added:

```sh
sudo apt-get update
sudo apt-get install wget
wget -O- https://mirror.oxfordnanoportal.com/apt/ont-repo.pub | sudo apt-key add -
echo "deb http://mirror.oxfordnanoportal.com/apt focal-stable non-free" | sudo tee /etc/apt/sources.list.d/nanoporetech.sources.list
```

I have yet to test these Focal packages, but it looks promising on my working system with the Bionic versions installed:

```sh
$ apt list --upgradable 
Listing... Done
minknow-core-minion-nc/stable 4.3.4-focal amd64 [upgradable from: 4.3.4-bionic]
ont-bream4-minion/stable,stable 6.2.5-1~focal all [upgradable from: 6.2.5-1~bionic]
ont-configuration-customer-minion/stable,stable 4.3.9-1~focal all [upgradable from: 4.3.9-1~bionic]
ont-jwt-auth/stable 0.28-1~focal amd64 [upgradable from: 0.28-1~bionic]
ont-kingfisher-ui-minion/stable,stable 4.3.22-1~focal all [upgradable from: 4.3.20-1~bionic]
```

The above is indicating that the 5 'core' ONT packages I have installed are wanting to be upgraded to their Focal equivalent (of the same version). I will test this when I get a chance and feed back. **To be clear, this is jumping the gun before ONT are ready and have announced that Focal is supported - so as always user beware, there be dragons!!**

-----

##### install MinION software

If you are running Ubuntu Bionic (18.04) the below should get the base ONT packages installed:

```sh
sudo apt-get update
sudo apt-get install minion-nc
```

If that works for you great! You can skip ahead to [this](https://github.com/sirselim/jetson_nanopore_sequencing/blob/main/live_basecalling.md#getting-the-correct-version-of-guppy) section and continue setting up Guppy GPU. If you are experiencing issues please read on.

As of the **MinKNOW 21.06.0** update the above cause's 'fun' dependency issues on Ubuntu 20.04 or other higher version distros. The issues result from the bundled Guppy packages in the `minion-nc` meta-package:

```sh
$ apt show minion-nc 
Package: minion-nc
Version: 21.06.0-1~bionic
Priority: optional
Section: science
Maintainer: Oxford Nanopore Technologies <info@nanoporetech.com>
Installed-Size: unknown
Depends: minknow-core-minion-nc (= 4.3.4-bionic),
         ont-bream4-minion (= 6.2.5-1~bionic),
         ont-configuration-customer-minion (= 4.3.9-1~bionic),
         ont-guppyd-for-minion (=5.0.11-1~bionic),
         ont-guppy-cpu-for-minion (=5.0.11-1~bionic),
         ont-kingfisher-ui-minion (= 4.3.20-1~bionic),
         ont-vbz-hdf-plugin
Breaks: minknow-nc (<< 19.02~~)
Replaces: minknow-nc (<< 19.02~~)
Download-Size: 3,438 B
APT-Sources: http://mirror.oxfordnanoportal.com/apt bionic-stable/non-free amd64 Packages
Description: ONT MinION meta-package for NC customers.
```

A ‘meta-package’ is a convenient way to bulk-install groups of applications, their libraries and documentation. Most of the time you don't need a meta-package, they are just suggestions of what the distributors believe you might want to install in bulk to deliver 'their recommended experience'.

In the above you can see that `ont-guppyd-for-minion` and `ont-guppy-cpu-for-minion` are wanting to be installed when installing `minion-nc`. The whole purpose of this exercise is to get Guppy GPU installed and working, so there is no need to bring those other versions of Guppy along for the ride. So why try to install them?

Thus, there is a 'manual' method to get around this. There are 5 packages that we need to install:

    minknow-core-minion-nc 
    ont-bream4-minion 
    ont-configuration-customer-minion 
    ont-kingfisher-ui-minion 
    ont-vbz-hdf-plugin

We can install these 5 packages without installing `minion-nc` like this:

```sh
sudo apt install minknow-core-minion-nc ont-bream4-minion ont-configuration-customer-minion ont-kingfisher-ui-minion ont-vbz-hdf-plugin
```

This may pull some other packages whilst installing, but you should no longer see dependency issues with the installation process and it should complete successfully. Now it's time to install Guppy GPU.

##### check version of software

```sh
apt policy minion-nc
```

You should see something like this as output of the above:

```sh
minion-nc:
  Installed: 21.06.0-1~xenial
  Candidate: 21.06.0-1~xenial
  Version table:
 *** 21.06.0-1~xenial 500
        500 http://mirror.oxfordnanoportal.com/apt xenial-stable/non-free amd64 Packages
        500 http://mirror.oxfordnanoportal.com/apt xenial-stable/non-free i386 Packages
        100 /var/lib/dpkg/status
```

If you are having issues with getting MinION software / MinKNOW setup I suggest you go to the community forum and search for similar questions/issues, and if nothing turns up you can create your own.

### Getting the correct version of Guppy

-----

**Quick Note:** I just want to reiterate here that we only really need one version of Guppy 'installed'/present on our system, the GPU version available via the Downloads page. This version of Guppy has been built against the required libraries (i.e. CUDA) to enable GPU acceleration of basecalling. While it's listed as the GPU version, it still has the ability to CPU basecall. I imagine the reason ONT don't just have one version is that if you don't need/want the GPU option then the CPU version will be a smaller download and install. But for our purposes we want to enable live basecalling with MinKNOW, so we want to utalise a powerful GPU, thus we want to grab this version. If we want to do CPU calling we can use the same Guppy, so there is no need to either download or keep older Guppy CPU versions around in my opinion. Arguably this is what may be causing some of the issues that people are seeing after the latest update (many people report that Guppy is working in CPU mode).

-----

This is something that seems to cause constant confusion, and I can see why! The versions of MinKNOW and Guppy are tightly tied together, **BUT** it doesn't mean that the latest version of each piece of software works with each other... are you with me so far?

What this means in practice is that currently the latest Linux version of MinKNOW for MinION is 21.06.0, which requires Guppy version 5.0.11. However, by the time you read this the latest and greatest version of Guppy is likely to be newer/higher. Now many people will be tracking that latest Guppy version as it's nice to have all the new bells and whistles, but it **will not** play nicely with MinKNOW and will crash out as soon as you try to run a sequencing experiment with live basecalling. There are ways to have multiple versions present on a system without causing issues, but more on that later.

For now the below table should help you decided which versions of software you need to grab:

| MinION Release | MinKnow Core version | GUI version | Guppy version working  |
|:--------------:|:--------------------:|-------------|------------------------|
|    20.06.5     |        4.0.5         | 4.0.21      | 4.0.9                  |
|    20.06.17    |        4.0.5         | 4.0.21      | 4.0.11, 4.0.14, 4.0.15 |
|    20.10.3     |        4.1.2         | 4.1.22      | 4.2.2, 4.2.3           |
|    21.02.1     |        4.2.5         | 4.2.8       | 4.3.4                  |
| 21.06.0 (21.05.8 MinIT) |	4.3.4 |	4.3.20 |	5.0.11 |

*the above is a snapshot of the latest software versions, if you require a more complete overview you can find that [here](https://github.com/sirselim/jetson_nanopore_sequencing#minknow--guppy-compatibility).

So, as stated above, the current MinION MinKNOW release is 21.06.0, and it needs Guppy 5.0.11

### Download the correct version of Guppy

This is where you are going to require access to the ONT community portal. Various pieces of ONT software are distributed (***freely***) from this portal, but only from this portal, i.e. they cannot be shared by third parties. So once you have access to the portal you can continue.

You should check out the software downloads page [here](https://community.nanoporetech.com/downloads), [this](https://community.nanoporetech.com/posts/enabling-gpu-basecalling-f) post, as well as [this](https://community.nanoporetech.com/posts/no-ampere-compatible-guppy) one. These will guide you in where and how to download the version of GPU Guppy that you will require.

Note: the last link above is a post that deals with issues that arose with the new Nvidia Ampere GPUs. This meant that versions of Guppy have had to be specially 'patched' for Ampere compatibility, however after Guppy 4.3.4 the pre-built versions available should work just fine.

### Extract the version of Guppy you downloaded

Once you have downloaded the correct (GPU) version of Guppy you can extract it:

```sh
tar -xzvf ont-guppy_5.0.11_linux64.tar.gz
```

The above example is for Guppy 5.0.11, but you can substitute for the compressed file of the version you downloaded.

This should result in a folder called `ont-guppy`, which if you run `ls` on should give something like this:

```sh
$ ls ont-guppy
bin  data  lib
```

The `bin` directory contains the binaries that we are interested in.

### Copy Guppy to `/opt/ont`

The below code assumes that you are currently in the directory where you extracted Guppy above. You should have the `ont-guppy` folder present at this level. The below will copy this directory and everything in it over to `/opt/ont`. You will likely need `sudo` (super user) access to the machine you are working on for this and many following steps.

|:exclamation: this will overwrite any version of Guppy that is present at this same location. Make sure this is what you want to do.|
|-------------------|

```sh
sudo cp -r ont-guppy /opt/ont/
```

### Create sym links to `/usr/bin`

Now we have copied `ont-guppy` and all it's contents this is where what I do differs from the suggested instructions. I create sym links of the newly copied binaries to `/usr/bin/` essentially making this version of Guppy (and in this case ONTs version of minimap2) the system-wide versions of these tools. Now this might not be for everyone, but I'm comfortable with what I am doing and this works well for me. As I stated above I will explain how I deal with other versions of Guppy later on.

-----

**An important note:** this step has been modified after the release of Guppy 4.2.2, which introduces `guppy_basecall_client` in place of the `guppy_basecaller`. This has a HUGE influence when we get to modifying the `app_conf` file soon.

This is the step that has been tripping a lot of people up lately, and is the major step that has not been updated in the ONT instructions.

**Warning:** I want to disclaim here that there is an official `config_editor` tool to make these changes in a more automated way than what I'm going to document. However, for whatever reason I've found the manual approach much more consistent while allowing me to fine tweak numerous settings whilst in the conf file - again your milage may vary.

If this is something you want to test the below is an example:

```sh
sudo /opt/ont/minknow/bin/config_editor --conf application --filename /opt/ont/minknow/conf/app_conf \
--set guppy.server_executable="/opt/ont/guppy/bin/guppy_basecall_server" \
--set guppy.client_executable="/opt/ont/guppy/bin/guppy_basecall_client" \
--set guppy.gpu_calling=1 \
--set guppy.num_threads=16 \
--set guppy.ipc_threads=2
```
Please note I have not tuned/optimised the above parameters, it's just the example ONT give. Also note that the defined paths are also those from the ONT example and not what I use for all my working MinKNOW installs.

-----

So on with the 'manual' approach...

If you have copied the downloaded GPU version of Guppy over to `/opt/ont/ont-guppy` then this code will create sym links of the binaries and make them accessible system wide:

```sh
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecaller /usr/bin/guppy_basecaller
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecall_client /usr/bin/guppy_basecall_client
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecall_server /usr/bin/guppy_basecall_server
sudo ln -s /opt/ont/ont-guppy/bin/guppy_basecaller_supervisor /usr/bin/guppy_basecaller_supervisor
sudo ln -s /opt/ont/ont-guppy/bin/guppy_barcoder /usr/bin/guppy_barcoder
sudo ln -s /opt/ont/ont-guppy/bin/guppy_aligner /usr/bin/guppy_aligner
sudo ln -s /opt/ont/ont-guppy/bin/minimap2 /usr/bin/minimap2
```

You should now be able to check to see if they are in fact being picked up by running the below:

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
: Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 5.0.11+2b6dbff, client-server API version 7.0.0
```

Notice that this is the correct version for MinION MinKNOW 21.06.0. Great!

### Modify MinKNOW `app_conf`

This is an extremely crucial step for the live GPU basecalling to actually work, possibly the most important. What we are going to do here is tell the `minknow` service running in the background exactly which Guppy (and minimap2) binaries it should be looking at and loading.

The file we're looking for is `app_conf`, it's located at this path: `/opt/ont/minknow/conf/app_conf`

Now this is again where I differ in my process. I modify this file manually as opposed to using a CLI configuration tool, in my opinion that that process is 'clunky' and it's easier using a text editor to make the changes. It also means that you can make changes to other Guppy related parameters while you're in there (i.e. you might want to adjust basecalling parameters based on your GPU) - discussion of these parameters is outside of this and I might talk about it further below.

So to modify `app_conf` I use a text editor such as `nano` but you can use whatever you wish.

My example is:

```sh
sudo nano /opt/ont/minknow/conf/app_conf
```

You will then need to navigate close to the bottom of the file, you're looking for the section that starts with:

```sh
"guppy": {
            "cereal_class_version": 0,
            "gpu_calling": true,
            "gpu_devices": "cuda:all",
            ...
```

The above is actually very important in terms of the parameters that indicate that we are in fact using GPU versions of Guppy binaries. So ensure yours looks like this.

In this section you will also notice various path variables being set, this is what we want to ensure is correct. In particular these paths:

```sh
            "client_executable": "/usr/bin/guppy_basecall_client",
            "barcoding_executable": "/usr/bin/guppy_barcoder",
            "alignment_executable": "/usr/bin/guppy_aligner",
            "server_executable": "/usr/bin/guppy_basecall_server",
```

We want these paths set to the location of our sym linked binaries, so the `/usr/bin/...` versions. Here is that relevant section of one of my systems `app_conf` files as an example of what you are looking at as a final result:

```sh
        "guppy": {
            "cereal_class_version": 0,
            "gpu_calling": true,
            "gpu_devices": "cuda:all",
            "server_port": 5555,
            "max_queued_reads": 5000,
            "max_client_queued_reads": 20000,
            "max_samples_in_flight": 50000000,
            "client_reconnect_attempts": 3,
            "max_refused_reads_before_restart": 5000,
            "num_threads": 4,
            "ipc_threads": 4,
            "gpu_runners_per_device": 48,
            "chunks_per_runner": 256,
            "client_executable": "/usr/bin/guppy_basecall_client",
            "barcoding_executable": "/usr/bin/guppy_barcoder",
            "alignment_executable": "/usr/bin/guppy_aligner",
            "server_executable": "/usr/bin/guppy_basecall_server",
            "config_file": "dna_r9.4.1_450bps_fast_prom.cfg",
            "log_path": {
                "value0": "/var/log/minknow/guppy"
            },
```

Don't be concerned that your `config_file` and various other variables are different, these are custom modifications I've made for the GPU on this machine (Nvidia Titan RTX). For now we just want to ensure we can get GPU basecalling running in MinKNOW live. Later on we can concentrate on optimising for speed and accuracy.

### Stop the MinKNOW service

Once you have made the above edits and saved the file you will need to restart the `minknow` service that's running in the background. You do that with the below:

```sh
sudo service minknow stop
```

### Confirm the guppy_basecall_server process isn't running

Run the below:

```sh
ps -A | grep guppy_basecall_
```

If the result of the above command is not blank, manually kill the process:

```sh
sudo killall guppy_basecall_server
```

### Start the MinKNOW service

If you are happy you can then start the `minknow` service again:

```sh
sudo service minknow start
```

You can then check it's status:

```sh
service minknow status
```

It should look something like this:

```sh
● minknow.service - MinKNOW Instrument Software for MinIT (daemon)
     Loaded: loaded (/etc/systemd/system/minknow.service; enabled; vendor preset: enabled)
     Active: active (running) since Tue 2021-03-23 14:28:04 NZDT; 2 weeks 0 days ago
   Main PID: 6283 (mk_manager_svc)
      Tasks: 311 (limit: 309008)
     Memory: 2.9G
        CPU: 4d 23h 36min 3.076s
     CGroup: /system.slice/minknow.service
             ├─ 6283 /opt/ont/minknow/bin/mk_manager_svc
             ├─ 6287 /opt/ont/minknow/bin/crashpad_handler --database=/share/minknow/data/core-dump-db --metrics-dir=/share/minknow/data/core-dump-db --url=https://submit.backtrace.io/nanoporetech/6e3c4958cfd996b7000dadb02a931ff700986aa48b33903b5902dff716c95809/minidump --annotation=cmd_line=/opt/ont/minknow/bin/mk_m>
             ├─ 6307 /usr/bin/guppy_basecall_server --config dna_r9.4.1_450bps_fast_prom.cfg --port 5555 --log_path /var/log/minknow/guppy --ipc_threads 4 --max_queued_reads 5000 --chunks_per_runner 256 --num_callers 4 -x cuda:all --gpu_runners_per_device 48
             ├─ 6308 /opt/ont/minknow/bin/grpcwebproxy --server_tls_cert_file=/opt/ont/minknow/conf/rpc-certs/localhost.crt --server_tls_key_file=/opt/ont/minknow/conf/rpc-certs/localhost.key --backend_addr=127.0.0.1:9501 --backend_tls_noverify --backend_tls=false --server_bind_address=0.0.0.0 --server_http_tls_port=>
             ├─ 6309 /opt/ont/minknow/bin/basecall_manager --guppy-address 5555 --conf /opt/ont/minknow/conf 0.0.0.0:9504
             ├─ 6321 /opt/ont/minknow/bin/crashpad_handler --database=/share/minknow/data/core-dump-db --metrics-dir=/share/minknow/data/core-dump-db --url=https://submit.backtrace.io/nanoporetech/6e3c4958cfd996b7000dadb02a931ff700986aa48b33903b5902dff716c95809/minidump --annotation=cmd_line=/opt/ont/minknow/bin/base>
             ├─ 6338 /opt/ont/minknow/bin/grpcwebproxy --server_tls_cert_file=/opt/ont/minknow/conf/rpc-certs/localhost.crt --server_tls_key_file=/opt/ont/minknow/conf/rpc-certs/localhost.key --backend_addr=127.0.0.1:9504 --backend_tls_noverify --backend_tls=false --server_bind_address=0.0.0.0 --server_http_tls_port=>
             ├─12793 /opt/ont/minknow/bin/control_server --config /tmp/minknow_7e3a11a1-8dda-43d4-9d7c-1c50131a4fce/conf-MN34702
             ├─12796 /opt/ont/minknow/bin/crashpad_handler --database=/share/minknow/data/core-dump-db --metrics-dir=/share/minknow/data/core-dump-db --url=https://submit.backtrace.io/nanoporetech/6e3c4958cfd996b7000dadb02a931ff700986aa48b33903b5902dff716c95809/minidump --annotation=cmd_line=/opt/ont/minknow/bin/cont>
             ├─12880 /opt/ont/minknow/bin/analyser --config /tmp/minknow_7e3a11a1-8dda-43d4-9d7c-1c50131a4fce/conf-MN34702 --shmem-prefix mk12793
             ├─12881 /opt/ont/minknow/bin/grpcwebproxy --server_tls_cert_file=/opt/ont/minknow/conf/rpc-certs/localhost.crt --server_tls_key_file=/opt/ont/minknow/conf/rpc-certs/localhost.key --backend_addr=127.0.0.1:8001 --backend_tls_noverify --backend_tls=true --server_bind_address=0.0.0.0 --server_http_tls_port=8>
             └─12889 /opt/ont/minknow/bin/crashpad_handler --database=/share/minknow/data/core-dump-db --metrics-dir=/share/minknow/data/core-dump-db --url=https://submit.backtrace.io/nanoporetech/6e3c4958cfd996b7000dadb02a931ff700986aa48b33903b5902dff716c95809/minidump --annotation=cmd_line=/opt/ont/minknow/bin/anal>
```

### Confirm that guppy_basecall_server is using the GPU

Another check is looking at the output of `nvidia-smi`

```sh
nvidia-smi
```

Mine looks like this:

```sh
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 460.67       Driver Version: 460.67       CUDA Version: 11.2     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  TITAN RTX           On   | 00000000:18:00.0 Off |                  N/A |
| 41%   31C    P8     4W / 280W |   6090MiB / 24215MiB |      0%      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|    0   N/A  N/A      1917      G   /usr/lib/xorg/Xorg                277MiB |
|    0   N/A  N/A      6307      C   ...bin/guppy_basecall_server     5725MiB |
|    0   N/A  N/A     15354      G   cinnamon                           27MiB |
|    0   N/A  N/A   1737263      G   /usr/lib/firefox/firefox            3MiB |
|    0   N/A  N/A   1737290      G   /usr/lib/firefox/firefox            3MiB |
|    0   N/A  N/A   1737322      G   /usr/lib/firefox/firefox            3MiB |
|    0   N/A  N/A   1737373      G   /usr/lib/firefox/firefox            3MiB |
|    0   N/A  N/A   1741095      G   ...gAAAAAAAAA --shared-files       13MiB |
|    0   N/A  N/A   3884069      G   ...AAAAAAAA== --shared-files       23MiB |
+-----------------------------------------------------------------------------+
```

You can see that GPU `guppy_basecall_server` is running. Hooray!

### Monitor your next run closely

Now hopefully you should be able to put in a flowcell and check that live GPU basecalling is working. It might be a good idea to use an old flowcell with a library already on it if you have something like that. Or you can put the device into playback/simulation mode, but that's not a 'simple' process - I'll leave it up to those people that want to dig a little deeper to explore this option.

For the first sequencing run you should be able to monitor using the MinKNOW GUI to make sure basecalling is working as expected. If it errors out make sure to check all the logs and error messages and see if you can pinpoint where things are going wrong.

Hopefully though this should get up up and running.

The below is extra topics I'd like to delve into but that aren't directly related to live basecalling as such.

-----

## Additional Guppy versions

The point of this section is that you can have multiple versions of Guppy downloaded and extracted and use these without breaking your live basecalling setup. This is useful if you want to recall data generated on the current setup with say the latest version of Guppy after the fact, i.e. you might want to use Guppy 5.0.11 on that data you generated and basecalled originally using 4.3.4,

This process is actually really straightforward. It really just involves pulling the specific pre-built binaries from ONT, extracting them somewhere and then running them locally. On a couple of my machines I have 12 or so different versions of Guppy sitting in a local space that I'm able to run and revisit particular things if required. Doing this means that there is no chance of conflict between the version required for MinKNOW to run live basecalling. 

### Overview

I just have a directory (/home/myuser/software/guppy/) where I download and extract the pre-built binaries. They can then be called either directly: 

* i.e. /home/myuser/software/guppy/4.4.2/bin/guppy_basecaller --version

 ... or you can create a specific sym link to somewhere on your `$PATH`:

* i.e. you can create a sym link and name the link in say `/usr/bin/` something like `guppy_basecaller_4.4.2` so that you could then just use it from the `$PATH` in any terminal.

Below is an example from one of my machines with output from the commands.

### Example from one of my MinION set ups

The below is an example of one of my machines which has many different versions of Guppy available. Here is an `ls` of the local directory:

```sh
~/software/guppy$ ls
3.1.5  3.2.4  3.3.0  3.3.3  3.4.1  3.4.3  4.2.2  4.2.2-CUDA11  4.3.4  4.4.1  4.4.2  4.5.2  5.0.7  5.0.11
```

All up that is **14** different versions of Guppy that I am able to use for local basecalling.

Here is a walk through using Guppy 4.4.2 as an example. As above it is extracted to a local directory. It can be run directly from this location:

```sh
# full path to binary
$ ~/software/guppy/4.4.2/ont-guppy/bin/guppy_basecaller --version
: Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 4.4.2+9623c16
```

We can also create sym links into our PATH. ***NOTE: it's important that the name is unique if you are doing this, you don't want to overwrite the link to the version MinKNOW 'sees'.***

```sh
# create sym link to /usr/bin
$ sudo ln -s ~/software/guppy/4.4.2/ont-guppy/bin/guppy_basecaller /usr/bin/guppy_basecaller_4.4.2
```

We can now call this version from any terminal without pointing directly at the local directory:

```sh
# run from the above sym linked binary
$ guppy_basecaller_4.4.2 --version
: Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 4.4.2+9623c16
```

You'll notice that the output should be exactly the same, if it's not something has gone wrong.

Now as a final check we can output the "MinKNOW version" once more:

```sh
$ guppy_basecaller --version
: Guppy Basecalling Software, (C) Oxford Nanopore Technologies, Limited. Version 4.3.4+ecb2805
```

So this all looks pretty good. There is limit to the number of Guppy versions you can have set up in this manner.

-----

## Changing GPU optimisation parameters

**... work in progress ...**

I have done a lot in this space and there is a lot of excellent knowledge online, so I want to try and capture some of that here. Check back soon!
