# jetson_nanopore_sequencing
A place to collate notes and resources of our journey into porting nanopore sequencing over to accessible, portable technology.

## files
A brief description of the included files in this repo. To understand more please look at the `setup-guide.txt` file.

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

* ?

## [this is still very much under construction]

In the mean time please feel free to look over various notes and presentations that we've put together:

  - \[**active community discussion**]: https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94
  - \[eResearch 2020 presentation]: https://sirselim.github.io/presentations/eResearch_2020/eResearch_presentation_livedemo_2020#1
  - \[initial Xavier notes/unboxing/setup]: https://hackmd.io/@Miles/HkumH7sBH
  - \[benchmarking guppy on various GPUs]: https://esr-nz.github.io/gpu_basecalling_testing/gpu_benchmarking.html

## Parts list (Hardware)

I've been asked about part's lists, what are we using, where do we get if from? So the below is an attempt to address this. If others have confirmed working hardware please feel free to make a PR/issue or similar.

**Note:** in the below I have highlighted which components are confirmed as working by our team.  
**Warning:** all prices are in New Zealand dollars unless otherwise stated. For us in New Zealand we need to import the Jetson boards, there are no local resellers.

**Main components**
The components listed below act as a replacement for a desktop computer or laptop to run MinKnow and interface with the ONT MinION. The benefit of the Nvidia Jetson family of *'single board computers'* is in their price and performance. The key feature being the onboard GPU, which, on the Xavier models at least, is more than able to keep up with live base calling a MinION flow cell. They also act as nice little headless base call servers.

* Nvidia Jetson Xavier NX / Xavier AGX (confirmed working by external collaborators on Jetson TX2 as well, but this board is starting to show it's age, Xavier NX isn't much more expensive for a large overall upgrade)
  * [**confirmed**] Jetson Xavier NX ([link](https://developer.nvidia.com/embedded/jetson-xavier-nx-devkit))
  * [**confirmed**] Jetson Xavier AGX ([link](https://developer.nvidia.com/embedded/jetson-agx-xavier-developer-kit))
  * Jetson TX2 ([link](https://developer.nvidia.com/embedded/jetson-tx2-developer-kit))
* NVMe solid state hard drive (you can go cheaper here, but a high speed drive does provide a little performance boost)
  * [**confirmed**] Samsung 970 EVO Plus 1TB M.2 (2280), NVMe SSD ([link](https://www.pbtech.co.nz/product/HDDSAM971601/Samsung-970-EVO-Plus-1TB-M2-2280NVMe-SSD-RWMax-350))
* micro SD card (needed for the Xavier NX, OS drive)
  * [**confirmed**] SanDisk 64GB Mobile Extreme Pro microSDXC ([link](https://www.pbtech.co.nz/product/MEMSDK3566/SanDisk-64GB-Mobile-Extreme-Pro-microSDXC-170MBS-r))

**Portability components**
Below is a list of what we are currently using to have a fully portable sequencing unit. There is obviously a wide range of components that can be mixed and matched

* touch screen
  * [**confirmed**] generic (no name?) 7 inch LCD 1024x600 HDMI touchscreen ([link](https://www.jaycar.co.nz/1024x600-hdmi-7in-screen-with-usb-capacitive-touch/p/XC9026))
* power pack / battery
  * [**confirmed**] RavPower 27000mAh 85W Power House Model: RP-PB055 ([link](https://www.ravpower.com/products/rp-pb055-27000mah-portable-charger))
* solar panel
  * [**confirmed**] Choetech 80W Foldable & Portable Solar Panel Charger with DC and USB Type Ports ([link](https://www.mightygadgets.co.nz/collections/solar-charger-1/products/copy-of-solar-charger-24w-portable-solar-panel-charger-with-dual-usb-ports-by-choetech))

## notes

* if you are looking for the `h5py` library compiled on the Xavier AGX using ONT's various software stack please download the zipped file found in this repository. You can find instructions for setting up [here](https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94#gistcomment-3481318).
