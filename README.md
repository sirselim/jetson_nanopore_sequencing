# jetson_nanopore_sequencing
A place to collate notes and resources of our journey into porting nanopore sequencing over to accessible, portable technology.

## files
A brief description of the included files in this repo. To understand more please look at the `setup-guide.txt` file.

* `setup-guide.txt`: a quick guide giving the basic steps to reproduce a running MinKnow environment on Jetson Xavier NX and AGX
* `nanoporetech.list`: ONT MinIT repository file to be placed in `/etc/apt/sources.list.d/`
* `ont-package-list-xavier.txt`: list of all ONT packages that need to be installed
* `minknow.service`: this is a systemd config file that correctly loads the minknow service as the root user

## [this is still very much under construction]

In the mean time please feel free to look over various notes and presentations that we've put together:

  - \[**active community discussion**]: https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94
  - \[eResearch 2020 presentation]: https://sirselim.github.io/presentations/eResearch_2020/eResearch_presentation_livedemo_2020#1
  - \[initial Xavier notes/unboxing/setup]: https://hackmd.io/@Miles/HkumH7sBH
  - \[benchmarking guppy on various GPUs]: https://esr-nz.github.io/gpu_basecalling_testing/gpu_benchmarking.html
  
## notes

* if you are looking for the `h5py` library compiled on the Xavier AGX using ONT's various software stack please download the zipped file found in this repository. You can find instructions for setting up [here](https://gist.github.com/sirselim/2ebe2807112fae93809aa18f096dbb94#gistcomment-3481318).
