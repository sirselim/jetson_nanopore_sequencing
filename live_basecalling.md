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