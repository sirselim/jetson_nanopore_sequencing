# Adaptive Sampling (ReadUntil) implemented on Nvidia Jetson Xavier AGX and NX

One thing that struck me as interesting and very exciting with all this work 'porting' the ONT software over to the Jetson boards is the fact that the adpative sampling option is in fact present in the MinKNOW UI. This is interesting because we are using the software derived for the MinIT - a device that is now discontinued and was never intended to have adaptive sampling. What is also interesting is that ONT have recently released a form of adaptive sampling on the Mk1c. It is slightly different, dubbed 'sketch' mode. Here is part of the announcement from ONT on this:

> * *A new 'sketch' model is used for basecalling small chunks to make decisions to eject or keep reads. This model provides low-latency basecalling, making the most of the available on-board compute.*
>* *Regular live basecalling during adaptive sampling is disabled for this beta release. This is to ensure that the best possible enrichment is reached by focusing on making sampling decisions. Once the adaptive sample experiment is complete, you will need to manually initiate post-run basecalling to call the enriched reads in the "pass" folder. We are currently looking at optimising our algorithms to re-enable live basecalling alongside adaptive sampling.* source: [link](https://community.nanoporetech.com/posts/beta-release-of-adaptive-s-7369)

The above is even more intriguing to me now on the back of what I have learnt from getting adaptive sampling up and running on the Xavier boards, but I'll touch on this later. For now here is how I managed to get things up and running and some of the 'fun' road blocks I hit along the way.

-----

**WARNING:** my usual disclaimer here that if you try this "at home"* you accept all risks and consequences of your actions.

(\*or at work or any other environment!)

-----

## Adaptive sampling is there! Sort of...

So I mentioned above that it was great and unexpected that the option for adaptive sampling is present on the Xavier AGX even using the MinIT repositories. This is present in the MinKNOW software that gets installed using the set up that I've detailed previously ([GitHub link](https://github.com/sirselim/jetson_nanopore_sequencing)) for getting up and running on Jetson boards.

So we thought we'd give it a crack. We prepared a library based on a Zymo mock community, which looks like this if you are interested:

| Species                  | Genomic DNA | Genome Copy | Cell Number |
|--------------------------|-------------|-------------|-------------|
| *Pseudomonas aeruginosa*   | 12          | 6.1         | 6.1         |
| *Escherichia coli*       | 12          | 8.5         | 8.5         |
| *Salmonella enterica*      | 12          | 8.7         | 8.8         |
| *Lactobacillus fermentum*  | 12          | 21.6        | 21.9        |
| *Enterococcus faecalis*    | 12          | 14.6        | 14.6        |
| *Staphylococcus aureus*    | 12          | 15.2        | 15.3        |
| *Listeria monocytogenes*   | 12          | 13.9        | 13.9        |
| *Bacillus subtilis*        | 12          | 10.3        | 10.3        |
| *Saccharomyces cerevisiae* | 2           | 0.57        | 0.29        |
| *Cryptococcus neoformans*  | 2           | 0.37        | 0.18        |

For fun I decided to try and enrich for one of the yeast species present at low levels (the Cryptococcus). So I created a minimap2 index for that genome and provided this in the appropriate section in MinKNOW, set up the rest of the run and waited to see what would happen. Here are some screen shots:

![](https://i.imgur.com/iU4h4jR.png)

![](https://i.imgur.com/0yjotzW.png)

![](https://i.imgur.com/3yOBBrG.png)

Hmm, adaptive sampling is "working" but it's taking a much longer time to make the decision on wither or not to unblock the read (purple peak). Notice the unblock peak mean is around 2.4Kb ish, we want this to be at about 450bp. Here is what that looks like on a system running the exact same experiment (same flowcell):

![](https://i.imgur.com/lhVYBf0.png)

![](https://i.imgur.com/bybtmzO.png)

A nice unblock peak at around 450bp, perfect. So what's going on with the Xavier AGX? Well, I had been troubleshooting some other ReadUntil/ReadFish work in playback/simulation mode where I was seeing similar results from both my x86/x64 Linux machine and the arm64 AGX. I had put it down to lack of 'fast' single core CPU performance and overhead from playback mode. BUT turning off live GPU basecalling in those situations actually made the world of difference, so what would happen if we did that here? More images (remember this is the 8 core, 16GB RAM version of the Xavier AGX):

![](https://i.imgur.com/zV9mMxl.png)

![](https://i.imgur.com/ivNkm4y.png)

Huh, what do you know, turn GPU live basecalling off and adaptive sampling works perfectly on the Xavier AGX. ***Slight aside:*** think back to what I mentioned earlier about the Mk1c, with it's beta 'sketch' mode - is this just adaptive sampling without live basecalling? That's one I'd love ONT to answer. :)

So it was extremely exciting to see this working, and working well. But monitoring with jtop I noticed that the GPU wasn't really even breaking a sweat and the CPU still had head room. Crazy idea, what would happen if we launch a background instance of guppy basecalling the data being generated? Picture time again:

![](https://i.imgur.com/BzG91sD.jpg)

The AGX handles this like a boss! So the Xavier AGX has the required horse power to both run adaptive sampling and GPU basecalling at the same time, but not when implemented at once via the MinKNOW UI. It appears that ONT have a lot of space for optimising the underlying code that handles how the reads are managed and delivered to the GPU on devices that might not be your 'standard fare' - something that they hinted at in the above summary of Mk1c adaptive sampling.

OK, this is really neat and exciting. We have a $699 USD computer that is able to not only keep up perfectly with live basecalling 1-2 MinIONs at once, it is also capable of adaptive sampling. Cool! But what about its little brother the Xavier NX? Time to find out...

## No love for the Xavier NX?

So time to boot up an NX, pop in the flowcell we've got the Zymo library loaded on, enter some run settings, tick adaptive sampling ... wait a minute, where is the option for adaptive sampling?!

The UI on the Xavier NX didn't provide me with the adaptive sampling tick box. I was sure both the AGX and NX were running the same software versions for everything, so I checked:

Xavier AGX
![](https://i.imgur.com/uzGHhl2.png)

Xavier NX
![](https://i.imgur.com/GO1uegV.png)

I also went right down to the `apt policy` level and confirmed the same package versions. I even went as far as caching the packages from the AGX, transferring them over to the NX, manuall installing each with `dpkg` and then being confused that the option still wasn't present in the UI. OK, puzzle solving time again!

### What is going on here?

I was actually quite stumped and the only thing that tickled at the edge of my mind was "what if they are doing some form of hardware polling under the hood" and then assigning a model/system to a machine based on it's hardware profile?

I remember seeing a particular package, `ont-system-identification`, being installed as part of the software that is pulled from the MinIT repositories. This seemed to put a series of scripts in `/opt/ont/platform/`, so it was time to dig into these.

Spoiler: my assumption was correct, that there is a level of hardware detection going on, which is then creating config files that effect 'downstream' components of the software 'stack' - at least on the official ONT devices this seems to be the case. I believe it's a series of scripts that are installed as part of the aptly named `ont-system-identification` package. This package appears to be a dependency for MinIT/Mk1C/GridION/PromethION installs, but not for Mk1b on general Linux OS. I did try removing it but it also wanted to nuke the rest of the MinKNOW install, not ideal...

When I first came across these scripts I tried running one called `ont-system-id` and got an error about a product code, and made the assumption that I probably didn't need to worry about it since I'm using a third party board/computer. This assumption, as seen below, was sort of correct (in the case of the Xavier AGX).

There is a particular script `system_details.py` (located here: `/opt/ont/platform/library/python/ont_system_identification/system_details.py`) that appears to be creating and matching specific hardware details, and in turn making a decision on the type of system that is being used.

So for example this is the read out of the config data generated for the Jetson Xavier AGX:

```shell
$ cat /etc/oxfordnanopore/configs/identity.config

Serial=esr-xavier
Fingerprint=
ProductCode=NONE
Class=NONE
Build=NONE
no_of_cores=8
no_of_gpus=3
gpu_type=NONE
memory=15G
no_of_disks=1
disk_technology=nvme
architecture=aarch64
```

My guess was that it's either setting the Xavier NX device as a MinIT or Mk1C based on the architecture, CPU cores and RAM details. As a reminder the Xavier NX has the same architecture (arm64), same CPU core count (6) and same amount of memory (8GB) as the Jetson TX2, which is inside the MinIT/Mk1c.

This turned out to be exactly what was happening, here is the output of the previously mentioned config file generated on the Xavier NX:

```shell
$ cat /etc/oxfordnanopore/configs/identity.config

Serial=xavnx01
Fingerprint=
ProductCode=MIN-101C
Class=MK1c
Build=proto-mk1c
no_of_cores=6
no_of_gpus=2
gpu_type=NONE
memory=7.6G
no_of_disks=1
disk_technology=nvme
architecture=aarch64
```

It can be clearly seen that the system identification scripts have decided that the Xavier NX looks similar enough to the Jetson TX2 inside the Mk1c to label it as such. So before trying anything 'fancy' I wondered what changing this config would do...

### 'Brute' forcing it...

Turns out that I had to use the `chattr` (change attribute) tool* to allow the file to be edited, I then made the modifications as per the AGX config file (replace "Mk1c" entries with "NONE"), saved the file, set the attributes back, restarted the minknow service and bingo bango adaptive sampling is now an option in the MinKNOW UI!

>\* _"chattr (Change Attribute) is a command line Linux utility that is used to set/unset certain attributes to a file in Linux system to secure accidental deletion or modification of important files and folders, even though you are logged in as a root user."_

These are the actual code steps that got me there:

```shell
#1. allow file to be edited
sudo chattr -i /etc/oxfordnanopore/configs/identity.config

#2. change lines in config file using nano
nano /etc/oxfordnanopore/configs/identity.config
# these are edited from mk1c or similar to NONE

#3. check the output
cat /etc/oxfordnanopore/configs/identity.config

# config file output
Serial=xavnx01
Fingerprint=
ProductCode=NONE
Class=NONE
Build=NONE
no_of_cores=6
no_of_gpus=2
gpu_type=NONE
memory=7.6G
no_of_disks=1
disk_technology=nvme
architecture=aarch64

#4. change attributes back, protecting the file
sudo chattr +i /etc/oxfordnanopore/configs/identity.config

#5. restart minknow service
sudo service minknow restart
```

And here is adaptive sampling working on a Xavier NX, enriching for Listeria: (again, live GPU basecalling is turned off)

![](https://i.imgur.com/LE6WtsK.png)

![](https://i.imgur.com/yiRUgtx.png)

![](https://i.imgur.com/AgyrTt9.png)

![](https://i.imgur.com/3wVf0ar.jpg)

## Additional thoughts

At the end of the day this is a pretty simple 'fix' to enable adaptive sampling using the software provided in the MinIT repositories. Additionally, I can see where other 'systems' can be added to the appropriate scripts that would fix this 'misidentification' issue - is this something that would be possible? i.e. would ONT ever explore adding various Jetson devices to the list? I'd love to have an answer to these questions, OR I would love to have the ability to make a pull request with the required changes! :)

It's great to have got adaptive sampling running on the Jetson Xavier family of devices, and it's exciting to see that there is potential head room, at least on the AGX, for live basecalling to be enabled again if the correct optimisations are made to the software by ONT. I'm very interested to see if ONT can get there with the Mk1c, if the older Jetson TX2 can pull it off it will be amazing! 

There are a couple of points of interest that arise here for me:

* (this one I've touched on) are ONT merely disabling live GPU basecalling currently on the Mk1c and calling this 'sketch' mode - that is my current hunch but I would love to have confirmation.
* if the above is true (even if it's not), the MinIT has the exact same compute hardware under the hood as the Mk1c and I have no doubt that it is capable of running adaptive sampling. I'm not suggesting that people with these devices go out and tinker with them, but I would love to get my hands on a MinIT and see how far it could be pushed - it could be at the very least made to run 'true' adaptive sampling with no live basecalling enabled - a 'feature' that I'm sure many MinIT owners would love to have (plus one that would save them a very expensive 'upgrade' fee to the Mk1c...).
* I'm also exploring the option of pairing multiple NX modules/boards together. It should be feasible to have one NX run MinKNOW with adaptive samping and another run Guppy basecall server, to which the live basecalling could be offloaded to.

Anyway, that's a bit more of a detailed run through of the journey I mentioned in the ONT community forum ([here](https://community.nanoporetech.com/posts/different-ui-but-using-sam) if you have access). I try to make as much as I can more freely available, hence these write ups that end up on GitHub, Gists or HackMD. Hopefully this has been helpful and/or an enjoyable read. As always, I'll update this document with anything in terms of new developments or corrections etc.

Thanks for reading.

-----

## After thoughts

I noted after the fact that the original script I 'played' with (`ont-system-id`) can actually be used to 'hard reset'/change the device to something else using the `--system` flag (see below). Since the Xavier AGX is "NONE" I need to try setting the NX to the same ("NONE") and hopefully we'll see the option for Adaptive Sampling magically appear in the MinKNOW UI - this would make the process I have explained above obsolete and would be much easier.

```shell
$ platform/bin/ont-system-id --help
Usage: ont-system-id [options]

Options:
  -h, --help            show this help message and exit
  -f FILE, --file=FILE  read config from FILE
  -c string, --code=string
                        Tell me details for a specific code
  -s SYSTEM, --system=SYSTEM
                        force system type
  -p, --print           Print system codes
  -q, --quiet           don't print status messages to stdout
```
