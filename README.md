[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4287657.svg)](https://doi.org/10.5281/zenodo.4287657)

|:exclamation: please be fore warned that by following this set up guide neither the authors or Oxford Nanopore Technology (ONT) are liable for any hardware/consumable damage or data loss.|
|-------------------|

|:memo: for more detailed information please feel free to browse the wiki pages [here](https://github.com/sirselim/jetson_nanopore_sequencing/wiki). |
|------------------------|

# Nvidia Jetson Nanopore Sequencing
A place to collate notes and resources of our journey into porting nanopore sequencing over to accessible, portable technology - Nvidia Jetson embedded computing.

![xavier-minknow](https://github.com/sirselim/jetson_nanopore_sequencing/raw/main/images/xavier-ont.gif)

(*click for video*)
[![NX1c](https://img.youtube.com/vi/Oik6zC2SET4/0.jpg)](https://youtu.be/Oik6zC2SET4)

**A sample of hardware images (for those so inclined)**

<img src="https://user-images.githubusercontent.com/5932864/97845231-372a2f00-1d51-11eb-8bd7-4a7f06ee536b.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845245-3c877980-1d51-11eb-90ca-8653a6c57666.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845252-3e513d00-1d51-11eb-8241-ac574b54d638.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845262-427d5a80-1d51-11eb-8f88-3e7de6df67e3.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845267-44471e00-1d51-11eb-8dfd-f47ce71f7877.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845269-44dfb480-1d51-11eb-90c8-8677766dee0f.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845279-47daa500-1d51-11eb-82f4-07ab69be756c.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845284-490bd200-1d51-11eb-8b5f-65d13d374497.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845286-49a46880-1d51-11eb-9700-8a222f86d76f.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845292-4a3cff00-1d51-11eb-83f9-7237f89a3fe4.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845304-4dd08600-1d51-11eb-9675-92ca6e52c620.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845307-4f01b300-1d51-11eb-8176-5b7eda5012ed.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845308-4f9a4980-1d51-11eb-9379-c0017ea7bef2.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845312-5032e000-1d51-11eb-9959-c1763b33bebf.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845318-51640d00-1d51-11eb-805f-9768047805df.jpg" width="23%"></img> <img src="https://user-images.githubusercontent.com/5932864/97845618-a869e200-1d51-11eb-91d6-885560bb1fc2.jpg" width="23%"></img> 

**...and some pictures of the final set up in action**

<img src="https://user-images.githubusercontent.com/5932864/97856233-e6223700-1d60-11eb-9e6b-efc2b4b454ff.png" width="30%"></img> <img src="https://user-images.githubusercontent.com/5932864/97855835-3baa1400-1d60-11eb-8c4b-d2c0e678537c.jpg" width="30%"></img> <img src="https://user-images.githubusercontent.com/5932864/97855887-52e90180-1d60-11eb-8bd2-c15c5683b219.jpg" width="30%"></img> <img src="https://user-images.githubusercontent.com/5932864/97856655-7496b880-1d61-11eb-85a5-e97d95c139e3.jpg" width="30%"></img> <img src="https://user-images.githubusercontent.com/5932864/97856727-909a5a00-1d61-11eb-9065-e739def23e16.png" width="30%"></img> <img src="https://user-images.githubusercontent.com/5932864/97855803-2fbe5200-1d60-11eb-8cd7-8bb1d6734e25.jpg" width="30%"></img> 

<img src="https://user-images.githubusercontent.com/5932864/102315158-8d3cf400-3fd8-11eb-8425-fa4e1511eed9.png" width="50%">

## What's this about?

**A little story**  
This project came about as a thought that I had whilst sitting in hospital waiting for test results from my son's lumbar puncture in late 2017. It ended up taking 48 hrs to return a negative result. At this stage I knew Nanopore sequencing was cheap and fast, and (back then) doable in hours. So why couldn't we easily get this sort of technology into hospitals? Why stop there, surely we could continue in the disruptive vein that ONT are paving and really democratise next-generation sequencing, providing it to the masses, think cheap 'off-the-shelf' parts that start to make this accessible to communities and developing countries. So that's what got the ball rolling, and it's been incredible seeing and meeting all the like-minded people on this journey to where we are now.

**Introducing the Nvidia Jetson embedded compute family**  
It wasn't until a year or so later that things really started to align. One factor was my increased involvement with nanopore data and realising the advantages of GPUs, the other was finding out about the Nvidia Jetson family. From Nvidia themselves:

> *"NVIDIA® Jetson™ systems provide the performance and power efficiency to run autonomous machines software, faster and with less power. Each is a complete System-on-Module (SOM), with CPU, GPU, PMIC, DRAM, and flash storage—saving development time and money. Jetson is also extensible. Just select the SOM that’s right for the application, and build the custom system around it to meet its specific needs."*  
> (source: [link](https://www.nvidia.com/en-us/autonomous-machines/embedded-systems/))

Knowing that these affordable but powerful (think Raspberry Pi on steroids) compute units were available, had Nvidia GPUs, and actually made up the 'heart' of the ONT MinIT (Jetson TX2), it was time to start exploring. There were a lot of bumps along the way (mainly due to the lack of ARM builds of various pieces of software), but long story short we are now at a point where MinKnow with live base calling works on the majority of the Jetson family. 

This has been greatly contribution to by numerous community members from around the globe, if you're interested in the in-depth development story told through gist comments check it out [here](https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94) - be ready for a long read! (no pun intended...)

## Getting started

Chances are that if you are here you are interested in setting up your own Jetson-based system. If so please read on. First this to note is that **this is still very much under construction** and will be continually developing. I'm aiming to create a more robust website to support the project, but for now this README will suffice.

### Where it started (paving the way)

Please feel free to look over various notes and presentations that we've put together over the last 12-18 months that directly support the current 'product':

- \[**active community discussion**]: https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94
- \[eResearch 2020 presentation]: https://sirselim.github.io/presentations/eResearch_2020/eResearch_presentation_livedemo_2020#1
- \[initial Xavier notes/unboxing/setup]: https://hackmd.io/@Miles/HkumH7sBH
- \[benchmarking guppy on various GPUs]: https://esr-nz.github.io/gpu_basecalling_testing/gpu_benchmarking.html

### Jetson Xavier AGX/NX specs

Here is a very quick overview of the 'compute' specs for each Xavier developer kit. Links to Nvidia provide much more detail.

#### Xavier AGX Dev Kit ([Nvidia link](https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit))

* **GPU** 	NVIDIA Volta architecture with 512 NVIDIA CUDA® cores and 64 Tensor cores  
* **CPU** 	8-core ARM v8.2 64-bit CPU, 8MB L2 + 4MB L3  
* **Memory** 	16GB/32GB 256-Bit LPDDR4x | 137GB/s  
* **Storage** 	32GB eMMC 5.1 [also has m.2 NVMe slot of expanded SSD option]  
* **Size** 	105 mm x 105 mm x 65 mm  

#### Xavier NX Dev Kit ([Nvidia link](https://developer.nvidia.com/embedded/jetson-xavier-nx-devkit))

* **GPU** 	NVIDIA Volta architecture with 384 NVIDIA CUDA® cores and 48 Tensor cores  
* **CPU** 	6-core NVIDIA Carmel ARM®v8.2 64-bit CPU 6 MB L2 + 4 MB L3  
* **Memory** 	8 GB 128-bit LPDDR4x @ 51.2GB/s  
* **Storage** 	microSD (not included) [also has m.2 NVMe slot of expanded SSD option]  
* **Size** 	103 mm x 90.5 mm x 34.66 mm 

### Parts list (Hardware)

I've been asked about part's lists, what are we using, where do we get if from? So the below is an attempt to address this. If others have confirmed working hardware please feel free to make a PR/issue or similar.

**Note:** in the below I have highlighted which components are confirmed as working by our team.  
**Warning:** all prices are in New Zealand dollars unless otherwise stated. For us in New Zealand we need to import the Jetson boards, there are no local resellers.

#### **Main components**  
The components listed below act as a replacement for a desktop computer or laptop to run MinKnow and interface with the ONT MinION. The benefit of the Nvidia Jetson family of *'single board computers'* is in their price and performance. The key feature being the onboard GPU, which, on the Xavier models at least, is more than able to keep up with live base calling a MinION flow cell. They also act as nice little headless base call servers.

* **Nvidia Jetson Xavier NX / Xavier AGX** (confirmed working by external collaborators on Jetson TX2 as well, but this board is starting to show it's age, Xavier NX isn't much more expensive for a large overall upgrade)  
  * Jetson Xavier NX ([link](https://developer.nvidia.com/embedded/jetson-xavier-nx-devkit)) [***confirmed***]
  * Jetson Xavier AGX ([link](https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit)) [***confirmed***]
  * Jetson TX2 ([link](https://developer.nvidia.com/embedded/jetson-tx2-developer-kit)) [***externally confirmed***]
* **NVMe solid state hard drive** (you can go cheaper here, but a high speed drive does provide a little performance boost)
  * Samsung 970 EVO Plus 1TB M.2 (2280), NVMe SSD ([link](https://www.pbtech.co.nz/product/HDDSAM971601/Samsung-970-EVO-Plus-1TB-M2-2280NVMe-SSD-RWMax-350)) [***confirmed***]
* **micro SD card** (needed for the Xavier NX, OS drive)
  * SanDisk 64GB Mobile Extreme Pro microSDXC ([link](https://www.pbtech.co.nz/product/MEMSDK3566/SanDisk-64GB-Mobile-Extreme-Pro-microSDXC-170MBS-r)) [***confirmed***]

#### **Portability components**  
Below is a list of what we are currently using to have a fully portable sequencing unit. This is ideally what you're wanting to add if you plan to take a MinION out into the field (from a compute perspective, wet-lab reagents and equipment are also required). There is obviously a wide range of components that can be mixed and matched here, but the below are confirmed compatible and working in our hands - see the above picture gallery for our set up.

* **touch screen**
  * generic (no name?) 7 inch LCD 1024x600 HDMI touchscreen ([link](https://www.jaycar.co.nz/1024x600-hdmi-7in-screen-with-usb-capacitive-touch/p/XC9026)) [***confirmed***]
* **power pack / battery**
  * RavPower 27000mAh 85W Power House Model: RP-PB055 ([link](https://www.ravpower.com/products/rp-pb055-27000mah-portable-charger)) [***confirmed***]
* **solar panel**
  * Choetech 80W Foldable & Portable Solar Panel Charger with DC and USB Type Ports ([link](https://www.mightygadgets.co.nz/collections/solar-charger-1/products/copy-of-solar-charger-24w-portable-solar-panel-charger-with-dual-usb-ports-by-choetech)) [***confirmed***]

### File description
A brief description of the included files in this repo. These are included to easy the initial installation and set up of the computing environment. To understand more please look at the `setup-guide.txt` file. This approach is currently confirmed working on both the Jetson Xavier NX and AGX systems, being replicated on multiple devices around the globe (New Zealand, Italy, Switzerland, USA).

**Available:**

* `setup-guide.txt`: a quick guide giving the basic steps to reproduce a running MinKnow environment on Jetson Xavier NX and AGX
* `nanoporetech.list`: ONT MinIT repository file to be placed in `/etc/apt/sources.list.d/`
* `ont-package-list-xavier.txt`: list of all ONT packages that need to be installed
* `minknow.service`: this is a systemd config file that correctly loads the minknow service as the root user
* `libs`: within this directory are precompiled versions of the `h5py` library. Most people will want to grab the python3.7 version, but the python2.7 version has been included as an option for 'legacy' versions of the MinKnow UI (now only possible if you have cached deb packages).

**Incoming (these files will be added eventually):**

* `user_conf`: a custom version of the user config file for MinKnow. This file contains the output path for data flow, this can be edited either in place (`/opt/ont/minknow/conf/`) or in this repo and then copied to the correct location.
* `app_conf`: a custom version of the app config file for MinKnow. This file contains the lots of configuration flags. The ones we're interested in mainly involve guppy basecalling parameters and paths to the binaries. This file can be edited either in place (`/opt/ont/minknow/conf/`) or in this repo and then copied to the correct location.

**Could be useful:**

* I'm looking into a script that might automate some of the above, i.e. take user input and perform the operations all at once to set up the likes of the conf files. Watch this space.

### notes and caveats

* if you are looking for the `h5py` library compiled on the Xavier AGX using ONT's various software stack please download the zipped file found in this repository. You can find instructions for setting up [here](https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94#gistcomment-3481318).

* **WARNING:** the current set up revolves around us leveraging the ONT MinIT ARM-based repositories. This process will only provided updated software for as long as the MinIT is supported by ONT, and now that it is discontinued it might be on a clock. Hopefully the software stack for the MinION Mk1c could be made to work in a similar fashion (they appear to still contain a Jetson TX2 at their heart). It's also extremely likely that ONT will release a new product based on the Xavier line, which should hopefully mean that we can then leverage that development. In the mean time it is recommended to download a cache of the currently installed packages to be able to rebuild a working system in the event that something like the repository being taken down, or an update from ONT breaking our efforts. To do this you can use a command such as `cat ont-package-list-xavier.txt | xargs sudo apt-get download` within the cloned repository. This assumes that you have set up the ONT Ubuntu Xenial MinIT repository. We are not making these packages available as they belong to ONT and this would break license agreements.
  * note: to cache the `deb` files without access to an arm64 device (Jetson board) you can do the following on a Linux computer (**WARNING:** only do this if you feel comfortable changing system architecture settings, otherwise wait until you have an arm64 device):
```sh
# add the ont minit repos
sudo echo "deb http://mirror.oxfordnanoportal.com/apt xenial-stable-minit non-free" > /etc/apt/sources.list.d/nanoporetech.mini.list
# add arm64 as an arch to your system
sudo dpkg --add-architecture arm64
# update to ensure you have the latest repos
sudo apt update
# from within git repo run this
cat ont-package-list-xavier.txt | xargs sudo apt-get download
# once packages have downloaded remove arm64 arch
sudo dpkg --remove-architecture arm64
# comment out/remove the ont minit repos
sudo rm /etc/apt/sources.list.d/nanoporetech.minit.list
# update and ensure everything is OK
sudo apt update
```

## optimising Guppy Base Calling on Jetson Xavier NX

General formula:

`runners * chunks_per_runner * chunk_size < 100'000 * [max GPU memory in GB] * 2`

### Xavier NX

So the current settings I am running on the Xavier NX are:

* --gpu_runners_per_device = **4** (*1 default MinIT config*)
* --chunks_per_runner = **256** (_48 default MinIT config_)
* --chunk_size = **1000** (_1000 default MinIT config_)

`4 * 256 * 1000 ~= 100'000 * 8 * 2` which is `1'024'000 < 1'600'000`

this is leaving some overhead for the likes of the OS and MinKnow.

Running with the default settings (as above) took **~5.3 mins** on a test data set. Running with the selected settings reduced this down to **~2.7 mins**.

Increasing the `chunks_per_runner` led to the largest gains in speed.

### Xavier AGX

The current 'optimal' settings I am running on the Xavier AGX are:

* --num_callers (num basecallers) = **2**
* --gpu_runners_per_device = **3** (*1 default MinIT config*)
* --chunks_per_runner = **1024** (_48 default MinIT config_)
* --chunk_size = **1000** (_1000 default MinIT config_)

`3 * 1024 * 1000 ~= 100'000 * 16 * 2` which is `3'072'000 < 3'200'000`

This is giving me a base calling rate of **~9.43x10<sup>6</sup> samples/s**.

## testing Readfish

Readfish is a software tool for selective sequencing (based on the read until API), more details [here](https://github.com/LooseLab/readfish/tree/dev).

```sh
# ensure you have a version of python3.7 with require dev libs
sudo apt install python3.7 python3.7-dev python3.7-venv
# Make a virtual environment
python3.7 -m venv readfish
. envs/readfish/bin/activate
pip install --upgrade pip
# install required packages/wheels/libs
pip install wheel
pip install Downloads/xavier_minit_build/ont_pyguppy_client_lib-4.0.15-cp37-cp37m-linux_aarch64.whl
pip install git+https://github.com/nanoporetech/read_until_api@v3.0.0
pip install git+https://github.com/LooseLab/readfish@dev
# check readfish is installed
readfish --version
```
Building minimap2 for arm:
```sh
git clone https://github.com/lh3/minimap2.git
cd minimap2
make arm_neon=1 aarch64=1
```

## MinKnow / Guppy compatibility

It's a rather unspoken topic, but the issues that arise between ONT software versions are rather 'fun' to deal with. The below table is a start at documenting the known versions of MinKnow, Kingfisher UI and Guppy that play nicely together on the Jetson Xavier.

| MinION Release          |  MinKnow Core version | GUI version           | Guppy version working  |
|:-----------------------:|:---------------------:|-----------------------|------------------------|
| 18.12.4                 | 3.1.8                 | 3.0.13                | 1.8.7                  |
| 18.12.6                 | 3.1.13                | 3.0.13                | 1.8.7                  |
| 18.12.9                 | 3.1.19                | 3.0.16                | 1.8.10                 |
| 19.05.0                 | 3.3.2                 | 3.3.16                | 3.0.3                  |
| 19.06.7                 | 3.4.5                 | 3.4.12                | 3.0.4                  | 
| 19.06.8                 | 3.4.8                 | 3.4.15                | 3.0.7                  |
| 19.10.1                 | 3.5.5                 | 3.5.10                | 3.2.6                  |
| 19.12.2                 | 3.6.0                 | 3.6.14                | 3.2.8                  | 
| 19.12.5                 | 3.6.5                 | 3.6.16                | 3.2.10                 |
| 20.06.4                 | 4.0.4                 | 3.5.10                | 4.0.9                  |
| 20.06.5                 | 4.0.5                 | 4.0.21                | 4.0.9                  |
| 20.06.17                | 4.0.5                 | 4.0.21                | 4.0.11, 4.0.14, 4.0.15 |
| 20.10.3                 | 4.1.2                 | 4.1.22                | 4.2.2, 4.2.3           |
| 21.02.1                 | 4.2.5                 | 4.2.8                 | 4.3.4                  |
| 21.06.0 (21.05.8 MinIT) | 4.3.4                 | 4.3.20                | 5.0.11                 |


* for earlier releases and a lot more information see [here](https://community.nanoporetech.com/downloads/minion_release/release_notes).
