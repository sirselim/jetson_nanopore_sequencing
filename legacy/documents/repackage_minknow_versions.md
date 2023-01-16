# Repackage MinKNOW versions from a current installation

## Background

This process was kindly contributed by [@hasindu2008](https://github.com/hasindu2008).

His original overview of the issue and solution:

> _"Recently me and [@Psy-Fer](https://github.com/Psy-Fer) encountered an issue where we wanted to install a particular version of MinKNOW on a new PC (aka the mini-fridge) for readfish and realised nanopore only provides the latest version. So we did some hacks and got MinKNOW on our old computer (aka the Fridge) transferred to the mini-fridge. Did not really expect it to work,  but it did - the magic power of dpkg package manager. Thought of documenting it somewhere, in case someone gets into similar trouble (and for myself later when I want to do it again) and thought this is the best place. So this is an enhancement rather than an issue."_

**On the old computer** 

1. `sudo apt install dpkg-repack`
2.  `apt-cache depends minion-nc |awk '{print $2}'` that prints the list of dependent packages

```
minknow-core-minion-nc
ont-bream4-minion
ont-configuration-customer-minion
ont-guppyd-for-minion
ont-guppy-cpu-for-minion
ont-kingfisher-ui-minion
ont-vbz-hdf-plugin
minknow-nc
minknow-nc
```

3.  Repack except the ont-guppyd-for-minion and ont-guppy-cpu-for-minion 

```
sudo dpkg-repack  minknow-core-minion-nc ont-bream4-minion ont-configuration-customer-minion ont-kingfisher-ui-minion ont-vbz-hdf-plugin
sudo dpkg-repack minknow-nc
```

4. copy the deb files to the new computer

**On the new computer** 

```
sudo dpkg -i minion-nc_20.10.3-1~bionic_all.deb
sudo dpkg -i ont-vbz-hdf-plugin_1.0.0-1~bionic_amd64.deb
sudo dpkg -i ont-kingfisher-ui-minion_4.1.22-1~bionic_all.deb
sudo apt-get install -f
sudo dpkg -i ont-configuration-customer-minion_4.1.15-1~bionic_all.deb
sudo dpkg -i ont-bream4-minion_6.1.4-1~bionic_all.deb
sudo dpkg -i *.deb
sudo apt-get install gconf-service gconf-service-backend gconf2 gconf2-common libappindicator1 libboost-atomic1.65.1 libboost-chrono1.65.1 libboost-log1.65.1 libboost-program-options1.65.1 libboost-regex1.65.1 libcurl4 libgconf-2-4 libindicator7 libnorm1 libpgm-5.2-0 libzmq5  ont-jwt-auth xvfb
```


