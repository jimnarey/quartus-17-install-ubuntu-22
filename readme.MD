## Quartus 17.1 and ModelSim 17.1 on Ubuntu 22

This document explains how to get the 17.1 versions of Quartus and ModelSim up and running on Ubuntu 22. These versions are needed to develop/contribute to cores for the [MiSTER FPGA project](https://boogermann.github.io/Bible_MiSTer/getting-started/introduction/).

### Problems

There are several problems you'll encounter when using the downloadable installers, without taking the addintional steps set out below.

- The installer hangs right at the end, when installing either the help or ModelSim components
- No menu item is created, once the previous problem is overcome
- Quartus 17.x requires libpng-12.so.0 which is no longer available in the Ubuntu repos
- Quartus throws 'Failed to load module "canberra-gtk-module"' when run (not sure how important this is, but certainly not fatal)
- Successfully installing Quartus means not installing ModelSim and it has to be installed separately
- Successfully installing Quartus means not installing the help files and these have to be installed separately, unless you're happy to use the online-only help
- ModelSim requires a set of 32bit pacakges and isn't as clear as it might be about what it needs
- ModelSim requires freetype 2.4.12 which is no longer available in the repos
- ModelSim requires 75dpi fonts to display correctly
- The startup script for ModelSim needs some adjustments to point it to some of the 32bit libraries and reflect the fact it isn't running on an ancient kernel.

Most of the the hard work to solve these problems was done by others. See 'References' below. I've added some additional steps needed because most of the slinked page are aimed at Ubuntu versions 16-18. I also wasn't able to find anywhere which had all the required steps in one place.

### Package Install

Enable 32bit (i386) architecture:

```
sudo dpkg --add-architecture i386
```

Install required 64bit and 32bit packages:

```
sudo apt install build-essential expat:i386 fontconfig:i386 g++-multilib gcc-multilib lib32gcc1 lib32stdc++6 lib32z1 libc6-dev libc6:i386 libcanberra0:i386 libexpat1:i386 libfreetype6:i386 libgtk-3-0:i386 libice6:i386 libncurses5:i386 libsm6:i386 libx11-6:i386 libxau6:i386 libxdmcp6:i386 libxext6:i386 libxft2 libxft2:i386 libxi6:i386 libxrender1:i386 libxt6:i386 libxtst6:i386 xfonts-75dpi zlib1g-dev zlib1g:i386
```

Enable source repos by:

- Opening 'Software' from the applications menu and checking the 'Source Code' box in the options (click the icon with three horizontal lines and then 'Software and Updates'); or
- Uncommenting each of the source repos in `/etc/apt/sources.list`

Install packages requiring source repos:

```
sudo apt build-dep -a i386 libfreetype6 
sudo apt install libc6-dev:i386
```

### Install Quartus 17.1

Download the Quartus 17.1 Linux installer from [the Intel website](https://www.intel.com/content/www/us/en/software-kit/669440/intel-quartus-prime-lite-edition-design-software-version-17-1-for-linux.html).

Extract the installation files with:

```
tar -xf Quartus-lite-17.1.0.590-linux.tar
```

This will extract three items: a `components` directory and two files, `readme.txt` and `setup.sh`.

Run `setup.sh`. Select which devices you want to be able to develop for (the DE-10 Nano board used by the MiSTER project uses the Cyclone V). **It is essential that the 'Help' and both 'ModelSim' components are deselected.** Selecting either will cause the installer to hang right at the end.

> The `setup.sh` script runs the `QuartusLiteSetup-17.1.0.590-linux.run` executable in the `components` directory. If you run `QuartusLiteSetup-17.1.0.590-linux.run --help` on the command line it lists a number of options which look promising in terms of successfully installing the Help and ModelSim components. None work. Any combination of options run from the desktop will cause the Help/ModelSim progress dialog to appear, which is what hangs, even the, `--mode text`, `--mode unattended` and `--unattendedmodeui none` options which, in any sane piece of software, would not cause diaglog boxes to appear at all. Running with these options under tty mode (e.g. via `Ctrl-Alt-F1`) causes the installer to hang altogether when it reaches 100%.

> Apparently the problem with the installer hanging can be overcome by changing window manager but I can confirm it arises under xfwm (Xubuntu) and i3 (Regolith). Given the frequency with which this issue has been raised on the Intel forums re Ubuntu, it's safe to assume it's also a problem under Gnome's default window manager.

The installation should then run normally but won't create menu items even if you select 'Create Desktop Shortcuts' when asked.

Once the installation is complete Quartus will run without any further steps (assuming you already installed the packages above) but obviously without ModelSim.

### Install ModelSim 17.1

The executable to install ModelSim can be found in the `components` directory extracted in the previous step. If you need to download it separately for any reason choose 'ModelSim-Intel® FPGAs Standard Edition Software Version 17.1' from [this page](https://www.intel.com/content/www/us/en/collections/products/fpga/software/downloads.html?edition=lite&platform=linux&download_manager=direct&product=modelsim_ae&s=Newest&p=10)

In a terminal enter the `components` directory and run:

```
./ModelSimSetup-17.1.0.590-linux.run --mode text
```

The default installation directory is `/home/user/intelFPGA/17.1` whereas Quartus installs to `/home/user/intelFPGA_lite/17.1` so you may want to change it to the latter.

The installer will appear to stall at 100% but just wait for it to complete. Once complete it will offer the option to offer feedback on the installer. The fact that selecting y/n really does hang the installer this time demonstrates, if nothing else, a keen sense of irony but it doesn't matter. The installation is complete. Press `Ctrl-C` to quit.

ModelSim will not run until the following steps have been completed.

### Install libping12

Enter/create a directory to download the source code into. A subdirectory will be created when you extract the libpng12 source.

Download and compile libpng12:

```
wget https://ppa.launchpadcontent.net/linuxuprising/libpng12/ubuntu/pool/main/libp/libpng/libpng_1.2.54.orig.tar.xz
tar Jxfv libpng_1.2.54.orig.tar.xz
cd libpng-1.2.54
./configure
make
sudo make install
```

Create symlinks to the newly installed shared objects:

```
sudo ln -s /usr/local/lib/libpng12.so.0.54.0 /usr/lib/libpng12.so
sudo ln -s /usr/local/lib/libpng12.so.0.54.0 /usr/lib/libpng12.so.0
```

### Install freetype 2.4.12

Download `freetype-2.4.12.tar.gz` from [sourceforge](https://sourceforge.net/projects/freetype/files/freetype2/2.4.12/) into the source code directory created/used above.

Extract the source code. It will create a subdirectory:

```
tar -xvf freetype-2.4.12.tar.gz
```

Compile freetype 2.4.12:

```
cd freetype-2.4.12
./configure --build=i686-pc-linux-gnu "CFLAGS=-m32" "CXXFLAGS=-m32" "LDFLAGS=-m32"
make -j8
```

Create a `lib32` directory in your ModelSim install directory. Depending on what you chose when you installed ModelSim (and whether you installed the free version) this will probably be `/home/user/intelFPGA/17.1/modelsim_ase` or `/home/user/intelFPGA_lite/17.1/modelsim_ase`. 

E.g. -

```
mkdir ~/intelFPGA_lite/17.1/modelsim_ase/lib32
```

Assuming you are still in the directory where you compiled freetype 2.4.12 run the following, changing the target directory to your newly created `lib32` directory as required:

```
sudo cp ./objs/.libs/libfreetype.so* ~/intelFPGA_lite/17.1/modelsim_ase/lib32
```

### Fixing the ModelSim startup script

Open the `vco` script found in the `modelsim_ase` directory using a text editor.

Search for the text `linux_rh60` (around line 210, depending on whether the file has already been edited) and change it to `linux`.

Replace the first uncommented line `mode=${MTI_VCO_MODE:-""}` with `mode=${MTI_VCO_MODE:-"32"}`.

Find the line ``` dir=`dirname "$arg0"` ``` at around line 50. Be careful searching, there are other (partial) matches for this text. Immediately after it, add the line:

```
export LD_LIBRARY_PATH=${dir}/lib32
```

ModelSim can now be run using:

```
.../modelsim_ase/bin/vsim
```

> `.../modelsim_ase/bin/vsim` is a symlink to the `.../modelsim_ase/vco` script edited in the previous steps. There are other `vsim` symlinks to this script, e.g. in `.../modelsim_ase/linuxaloem` but these throw errors when run. Use the symlink in the `bin` subdirectory.

### Point Quartus to ModelSim

Open Quartus and choose `Tools > Options` from the menu bar. In the options dialog click on `EDA Tool Options`. In the `ModelSim-Altera` field enter the full path to your `.../modelsim_ase/bin` directory, e.g. `/home/user/intelFPGA/17.1/modelsim_ase/bin`. 

You can now run ModelSim from within Quartus.

### Install the Quartus help (local)

It's not necessary to install a local copy of the Quartus Help if you are happy to use the online copy. This is opened from the 'Help' menu in Quartus if the help files are not installed locally. It's also worth noting that Chrome restricts the usage of JavaScript by local files and throws some warnings to this effect when viewing the help locally (it's not clear how much of a problem this is, they seem to display OK).

If you do want to install the help files, enter the `components` directory extracted when installing Quartus and run:

```
./QuartusHelpSetup-17.1.0.590-linux.run --mode text
```

Naturally, the installer will hang at the point it says `Setup has finished installing Quartus Prime Help` but the help files have been installed successfully by this point and the installer can be safely closed with `Ctrl-C`.

Now when you click on items under the 'Help' menu in Quartus the local copy will be loaded.

### References

- [aristocrates/ModelSim 17.1 for (Gentoo) Linux.md](https://gist.github.com/aristocrates/ea579917adfcca5751c69b28d84bbae2)

- [Install libpng12 on ubuntu 22.04](https://askubuntu.com/questions/1404213/install-libpng12-on-ubuntu-22-04)

- [Software installation in Linux](https://linux-software-list.readthedocs.io/en/latest/ubuntu.html#quartus-modelsim)

- [PrieureDeSion/modelsim_installation.md](https://gist.github.com/PrieureDeSion/e2c0945cc78006b00d4206846bdb7657)