## Setup half-life on Raspberry Pi or Jetson Nano.
---

There are a few ways to install:
1. There HL Source engine [leak](https://github.com/nillerusr/source-engine) and there's this [release](https://github.com/ValveSoftware/halflife?tab=readme-ov-file)
2. There's an open-source engine Xash3d, which emulates gold-source engine.
We'll go the second way.

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

The engine is now finished installing. Now we need game files ... ;)

Try this for ISO disc images: https://archive.org/download/half-life-generation_202401. At the time of writing, this had all three: HL, OpFor, Bshift.

There could be `.pak` files or `.wad` files that need to be extracted, and here's the friendly tool for all the extracting needs: https://github.com/RavuAlHemio/hllib

Install it and extract to your hearts content.

### Appendix
This guy followed the first process: [https://www.youtube.com/watch?v=LvWDqrgSo7Y&t=1s](https://youtu.be/LvWDqrgSo7Y)
