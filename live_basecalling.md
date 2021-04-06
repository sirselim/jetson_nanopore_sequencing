|:exclamation: please be fore warned that by following this set up guide neither the authors or Oxford Nanopore Technology (ONT) are liable for any hardware/consumable damage or data loss.|
|-------------------|

# Setting up GPU live basecalling

## Caveats

* **GPU basecalling is supported on Linux and NVIDIA GPUs only.**
  * The GPU must be installed on the same system to which the MinION is connected.
    * This is done at the user's own risk: mis-configuration of the GPU may result in slow basecalling and/or a large number of skipped reads.
      * i.e. if the basecall server crashes as a result of mis-parameterization. 
* GPUs with a low amount of memory (**less than 4 GB**) may not work at all.