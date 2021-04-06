|:exclamation: please be fore warned that by following this set up guide neither the authors or Oxford Nanopore Technology (ONT) are liable for any hardware/consumable damage or data loss.|
|-------------------|

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
$ ls ont-guppy
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

Now we have copied `ont-guppy` and all it's contents this is where what I do differs from the suggested instructions. I create sym links of the newly copied binaries to `/usr/bin/` essentially making this version of Guppy (and in this case ONTs version of minimap2) the system-wide versions of these tools. Now this might not be for everyone, but I'm comfortable with what I am doing and this works well for me. As I stated above I will explain how I deal with other versions of Guppy later on.

-----

**An important note:** this step has been modified after the release of Guppy 4.2.2, which introduces `guppy_basecall_client` in place of the `guppy_basecaller`. This has a HUGE influence when we get to modifying the `app_conf` file soon.

This is the step that has been tripping a lot of people up lately, and is the major step that has not been updated in the ONT instructions.

-----

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
## Additional Guppy versions

**... work in progress ...**

The point of this section is that you can have multiple versions of Guppy downloaded and extracted and use these without breaking your live basecalling setup. This is useful if you want to recall data generated on the current setup with say the latest version of Guppy after the fact, i.e. you might want to use Guppy 4.5.2 on that data you generated and basecalled originally using 4.3.4,

I'll try and update this asap. Please check back soon.

## Changing GPU optimisation parameters

**... work in progress ...**

I have done a lot in this space and there is a lot of excellent knowledge online, so I want to try and capture some of that here. Check back soon!