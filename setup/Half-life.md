## Setup half-life on Raspberry Pi or Jetson Nano.
---

There are a few ways to install:
1. There HL Source engine [leak](https://github.com/nillerusr/source-engine) plus this [release](https://github.com/ValveSoftware/halflife?tab=readme-ov-file)
2. There's an open-source engine Xash3d, which emulates gold-source engine. We'll go this way.

If you're interested in the first process, here's a [video](https://youtu.be/LvWDqrgSo7Y)

## Install Xash3D source first
Exexute these commands
```
sudo apt install git build-essential python libsdl2-dev libfreetype6-dev libopus-dev libbz2-dev libvorbis-dev libopusfile-dev libogg-dev
git clone --recursive https://github.com/FWGS/xash3d-fwgs
cd xash3d-fwgs
./waf configure
./waf build
./waf install --destdir=/path/to/output_dir
```

The engine is now finished installing. 

Now we need game files. This game is the greatest of all time, you can buy it, on steam. Get the datafiles from the installation folder once you have installed it locally on you windows machine or mac or linux or wherever Steam is supported. 

If you only have jetson to play with, well Archive.org is your friend. Try this for ISO disc images: https://archive.org/download/half-life-generation_202401. At the time of writing, this had all three: HL, OpFor, Bshift. Use the tools given in appendix to extract files.

## Procedure for MODs like Opposing Force and BlueShift

First copy the datafiles (`opfor` or `bshift`) to xash3d folder 
Now you are only missing the platform specific libs (`*.so`) in folders `cl_dlls/` and `dlls/` folder (which contain dlls and are worth fukall).

So now, download a repo that will build these for us (take a bow to the mightly folks at Xash3D)
Do this for Opposing force (checkout `opforfixed` branch or whichever is latest)
```
git clone --recursive https://github.com/FWGS/hlsdk-portable.git
cd hlsdk-portable
git checkout opforfixed  # Switch to the Opposing Force branch
```

OR blushift
```
git clone --recursive https://github.com/FWGS/hlsdk-portable.git
cd hlsdk-portable
git checkout bshift
```

And fire the build:
```
mkdir build
cd build
cmake ..
make -j4
```

You'll find some (smoking) dlls in `build/cl_dlls` and `build/dlls` folders. Copy them to respective folders inside Xash3D installation.

You should now be able to play the game by `xash3d -game <folder/name/in/xash3d>`.

To activate debug logs, use `xash3d -dev 2 -game <folder>`

### Appendix
---
#### Some tools of the trade
There could be `.pak` files or `.wad` files that need to be extracted, and here's the friendly tool for all the extracting needs: https://github.com/RavuAlHemio/hllib
Install it and extract to your hearts content.
