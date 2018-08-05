---
title: "Compile VLC with Hardware Acceleration for the Raspberry Pi (Stretch)"
date: 2018-08-05T23:35:46+02:00
draft: false
tags: [ "VLC", Raspberry Pi" ]
---

# Compile VLC with Hardware Acceleration for the Raspberry Pi (Stretch)

Credits to [Raspberry Pi Forum](https://www.raspberrypi.org/forums/viewtopic.php?t=195221).

The VLC version from Raspbian doesn't support the GPU of the RPi, and isn't able to play high resolution videos. An alternative is to compile to include this, but be careful as depending on the Raspbian distribution (Stretch, Wheezy, etc.) the dependencies and patches may be different.  This notebook targets Stretch, and it has been well-tested in a Raspberry Pi version 3.

```bash
sudo apt-get install autopoint pkg-config libtool build-essential autoconf
```

And more to install:

```bash
sudo apt-get install liba52-0.7.4 libasound2 libass5 libavahi-client3 libavahi-common3 libavc1394-0 libbasicusageenvironment1 libbluray1 libbz2-1.0 libc6 libcairo2 libcddb2 libcdio13 libchromaprint1 libdbus-1-3 libdc1394-22 libdca0 libdirectfb-1.2-9 libdvbpsi10 libdvdnav4 libdvdread4 libebml4v5 libfaad2 libflac8 libfontconfig1 libfreetype6 libfribidi0 libgcc1 libgcrypt20 libglib2.0-0 libgme0 libgnutls30 libgpg-error0 libgroupsock8 libgsm1 libjpeg62-turbo libkate1 liblirc-client0 liblivemedia57 liblua5.2-0 liblzma5 libmad0 libmatroska6v5 libmodplug1 libmp3lame0 libmpcdec6 libmpeg2-4 libmtp9 libncursesw5 libogg0 libopus0 libpng16-16 libpulse0 libraw1394-11 libresid-builder0c2a librsvg2-2 librtmp1 libsamplerate0 libsdl-image1.2 libsdl1.2debian libshine3 libshout3 libsidplay2 libsnappy1v5 libsndio6.1 libspeex1 libspeexdsp1 libssh-gcrypt-4 libssh2-1 libstdc++6 libtag1v5 libtheora0 libtinfo5 libtwolame0 libudev1 libupnp6 libusageenvironment3 libva-drm1 libva-x11-1 libva1 libvcdinfo0 libvorbis0a libvorbisenc2 libvpx4 libwavpack1 libwebp6 libwebpmux2 libx11-6 libx264-148 libx265-95 libxcb-keysyms1 libxcb1 libxml2 libxvidcore4 libzvbi0 zlib1g libgdk-pixbuf2.0-0 libgtk2.0-0 libnotify4 libqt5core5a libqt5gui5 libqt5widgets5 libqt5x11extras5 libxi6 libsmbclient libxext6 libxinerama1 libxpm4 fonts-freefont-ttf libaa1 libcaca0 libegl1-mesa libgl1-mesa-glx libgles1-mesa libgles2-mesa libxcb-shm0 libxcb-xv0 libxcb-randr0 libxcb-composite0
```

And even more to install (development packages):

```bash
sudo apt-get install liba52-0.7.4-dev libasound2-dev libass-dev libavahi-client-dev libavc1394-dev libbluray-dev libbz2-dev libc6-dev libcairo2-dev libcddb2-dev libcdio-dev libchromaprint-dev libdbus-1-dev libdc1394-22-dev libdca-dev libdirectfb-dev libdvbpsi-dev libdvdnav-dev libdvdread-dev libebml-dev libfaad-dev libflac-dev libfontconfig1-dev libfreetype6-dev libfribidi-dev libgcc-6-dev libgcrypt20-dev libglib2.0-dev libgme-dev libgnutls28-dev libgpg-error-dev libgsm1-dev libjpeg62-turbo-dev libkate-dev liblircclient-dev liblivemedia-dev liblua5.2-dev liblzma-dev libmad0-dev libmatroska-dev libmodplug-dev libmp3lame-dev libmpcdec-dev libmpeg2-4-dev libmtp-dev libncursesw5-dev libogg-dev libopus-dev libpng-dev libpulse-dev libraw1394-dev libresid-builder-dev librsvg2-dev librtmp-dev libsamplerate0-dev libsdl-image1.2-dev libsdl1.2-dev libshine-dev libshout3-dev libsidplay2-dev libsnappy-dev libsndio-dev libspeex-dev libspeexdsp-dev libssh-gcrypt-dev libssh2-1-dev libstdc++-6-dev libtag1-dev libtheora-dev libtinfo-dev libtwolame-dev libudev-dev libupnp6-dev libva-dev libvcdinfo-dev libvorbis-dev libvpx-dev libwavpack-dev libwebp-dev libx11-dev libx264-dev libx265-dev libxcb-keysyms1-dev libxcb1-dev libxml2-dev libxvidcore-dev libzvbi-dev zlib1g-dev libgdk-pixbuf2.0-dev libgtk2.0-dev libnotify-dev libqt5x11extras5-dev libxi-dev libsmbclient-dev libxext-dev libxinerama-dev libxpm-dev libaa1-dev libcaca-dev libegl1-mesa-dev libgles1-mesa-dev libgles2-mesa-dev libxcb-shm0-dev libxcb-xv0-dev libxcb-randr0-dev libxcb-composite0-dev libavcodec-dev libavformat-dev libgstreamer1.0-dev libswscale-dev
```

The VLC sources to compile:

```bash
wget https://download.videolan.org/vlc/2.2.8/vlc-2.2.8.tar.xz
tar -xJf vlc-2.2.8.tar.xz
```

And the patch:

```bash
wget http://steinerdatenbank.de/software/vlc-2.2.8-ffmpeg3-1.patch
```

Prepare for compiling:

```bash
cd vlc-2.2.8
./bootstrap
patch -Np1 -i ../vlc-2.2.8-ffmpeg3-1.patch
sed -i 's/error-implicit-function-declaration//' configure
```

Set the variables for compiling:

```bash
export CFLAGS="-I/opt/vc/include/ -I/opt/vc/include/interface/vcos/pthreads -I/opt/vc/include/interface/vmcs_host/linux -I/opt/vc/include/interface/mmal -I/opt/vc/include/interface/vchiq_arm -I/opt/vc/include/IL -I/opt/vc/include/GLES2 -I/opt/vc/include/EGL -mfloat-abi=hard -mcpu=cortex-a7 -mfpu=neon-vfpv4" CXXFLAGS="-I/opt/vc/include/ -I/opt/vc/include/interface/vcos/pthreads -I/opt/vc/include/interface/vmcs_host/linux -I/opt/vc/include/interface/mmal -I/opt/vc/include/interface/vchiq_arm -I/opt/vc/include/IL -mfloat-abi=hard -I/opt/vc/include/GLES2 -I/opt/vc/include/EGL -mcpu=cortex-a7 -mfpu=neon-vfpv4" LDFLAGS="-L/opt/vc/lib"
```

Configure for compiling:

```bash
./configure --prefix=/usr --enable-omxil --enable-omxil-vout --enable-rpi-omxil --disable-mmal-codec --disable-mmal-vout --enable-gles2
```

After some minutes:

```bash
make -j3
```

And if everything goes OK, after a long wait:

```bash
sudo make install
```