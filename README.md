# valgrind_3.24.0


How to cross-compile and run on Android.  Please read to the end,
since there are important details further down regarding crash
avoidance and GPU support.

These notes were last updated on 4 Nov 2014, for Valgrind SVN
revision 14689/2987.

These instructions are known to work, or have worked at some time in
the past, for:

arm:
  Android 4.0.3 running on a (rooted, AOSP build) Nexus S.
  Android 4.0.3 running on Motorola Xoom.
  Android 4.0.3 running on android arm emulator.
  Android 4.1   running on android emulator.
  Android 2.3.4 on Nexus S worked at some time in the past.

x86:
  Android 4.0.3 running on android x86 emulator.

mips32:
  Android 4.1.2 running on android mips emulator.
  Android 4.2.2 running on android mips emulator.
  Android 4.3   running on android mips emulator.
  Android 4.0.4 running on BROADCOM bcm7425

arm64:
  Android 4.5 (?) running on ARM Juno

On android-arm, GDBserver might insert breaks at wrong addresses.
Feedback on this welcome.

Other configurations and toolchains might work, but haven't been tested.
Feedback is welcome.

Toolchain:

  For arm32, x86 and mips32 you need the android-ndk-r6 native
    development kit.  r6b and r7 give a non-completely-working build;
    see http://code.google.com/p/android/issues/detail?id=23203
    For the android emulator, the versions needed and how to install
    them are described in README.android_emulator.

    You can get android-ndk-r6 from
    http://dl.google.com/android/ndk/android-ndk-r6-linux-x86.tar.bz2

  For arm64 (aarch64) you need the android-ndk-r10c NDK, from
    http://dl.google.com/android/ndk/android-ndk-r10c-linux-x86_64.bin

Install the NDK somewhere.  Doesn't matter where.  Then:


# Modify this (obviously).  Note, this "export" command is only done
# so as to reduce the amount of typing required.  None of the commands
# below read it as part of their operation.
#
export NDKROOT=/path/to/android-ndk-r<version>


# Then cd to the root of your Valgrind source tree.
#
cd /path/to/valgrind/source/tree


# After this point, you don't need to modify anything.  Just copy and
# paste the commands below.


# Set up toolchain paths.
#
# For ARM
export AR=$NDKROOT/toolchains/arm-linux-androideabi-4.4.3/prebuilt/linux-x86/bin/arm-linux-androideabi-ar
export LD=$NDKROOT/toolchains/arm-linux-androideabi-4.4.3/prebuilt/linux-x86/bin/arm-linux-androideabi-ld
export CC=$NDKROOT/toolchains/arm-linux-androideabi-4.4.3/prebuilt/linux-x86/bin/arm-linux-androideabi-gcc

# For x86
export AR=$NDKROOT/toolchains/x86-4.4.3/prebuilt/linux-x86/bin/i686-android-linux-ar
export LD=$NDKROOT/toolchains/x86-4.4.3/prebuilt/linux-x86/bin/i686-android-linux-ld
export CC=$NDKROOT/toolchains/x86-4.4.3/prebuilt/linux-x86/bin/i686-android-linux-gcc

# For MIPS32
export AR=$NDKROOT/toolchains/mipsel-linux-android-4.8/prebuilt/linux-x86_64/bin/mipsel-linux-android-ar
export LD=$NDKROOT/toolchains/mipsel-linux-android-4.8/prebuilt/linux-x86_64/bin/mipsel-linux-android-ld
export CC=$NDKROOT/toolchains/mipsel-linux-android-4.8/prebuilt/linux-x86_64/bin/mipsel-linux-android-gcc

# For ARM64 (AArch64)
export AR=$NDKROOT/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-ar 
export LD=$NDKROOT/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-ld
export CC=$NDKROOT/toolchains/aarch64-linux-android-4.9/prebuilt/linux-x86_64/bin/aarch64-linux-android-gcc


# Do configuration stuff.  Don't mess with the --prefix in the
# configure command below, even if you think it's wrong.
# You may need to set the --with-tmpdir path to something
# different if /sdcard doesn't work on the device -- this is
# a known cause of difficulties.

# The below re-generates configure, Makefiles, ...
# This is not needed if you start from a release tarball.
./autogen.sh

# for ARM
CPPFLAGS="--sysroot=$NDKROOT/platforms/android-3/arch-arm" \
   CFLAGS="--sysroot=$NDKROOT/platforms/android-3/arch-arm" \
   ./configure --prefix=/data/local/Inst \
   --host=armv7-unknown-linux --target=armv7-unknown-linux \
   --with-tmpdir=/sdcard
# note: on android emulator, android-14 platform was also tested and works.
# It is not clear what this platform nr really is.

# for x86
CPPFLAGS="--sysroot=$NDKROOT/platforms/android-9/arch-x86" \
   CFLAGS="--sysroot=$NDKROOT/platforms/android-9/arch-x86 -fno-pic" \
   ./configure --prefix=/data/local/Inst \
   --host=i686-android-linux --target=i686-android-linux \
   --with-tmpdir=/sdcard

# for MIPS32
CPPFLAGS="--sysroot=$NDKROOT/platforms/android-18/arch-mips" \
   CFLAGS="--sysroot=$NDKROOT/platforms/android-18/arch-mips" \
   ./configure --prefix=/data/local/Inst \
   --host=mipsel-linux-android --target=mipsel-linux-android \
   --with-tmpdir=/sdcard

# for ARM64 (AArch64)
CPPFLAGS="--sysroot=$NDKROOT/platforms/android-21/arch-arm64" \
   CFLAGS="--sysroot=$NDKROOT/platforms/android-21/arch-arm64" \
   ./configure --prefix=/data/local/Inst \
   --host=aarch64-unknown-linux --target=aarch64-unknown-linux \
   --with-tmpdir=/sdcard


# At the end of the configure run, a few lines of details
# are printed.  Make sure that you see these two lines:
#
# For ARM:
#          Platform variant: android
#     Primary -DVGPV string: -DVGPV_arm_linux_android=1
#
# For x86:
#          Platform variant: android
#     Primary -DVGPV string: -DVGPV_x86_linux_android=1
#
# For mips32:
#          Platform variant: android
#     Primary -DVGPV string: -DVGPV_mips32_linux_android=1
#
# For ARM64 (AArch64):
#          Platform variant: android
#     Primary -DVGPV string: -DVGPV_arm64_linux_android=1
#
# If you see anything else at this point, something is wrong, and
# either the build will fail, or will succeed but you'll get something
# which won't work.


# Build, and park the install tree in `pwd`/Inst
#
make -j4
make -j4 install DESTDIR=`pwd`/Inst


# To get the install tree onto the device:
# (I don't know why it's not "adb push Inst /data/local", but this
# formulation does appear to put the result in /data/local/Inst.)
#
adb push Inst /


# To run (on the device).  There are two things you need to consider:
#
# (1) if you are running on the Android emulator, Valgrind may crash
#     at startup.  This is because the emulator (for ARM) may not be
#     simulating a hardware TLS register.  To get around this, run
#     Valgrind with:
#       --kernel-variant=android-no-hw-tls
# 
# (2) if you are running a real device, you need to tell Valgrind
#     what GPU it has, so Valgrind knows how to handle custom GPU
#     ioctls.  You can choose one of the following:
#       --kernel-variant=android-gpu-sgx5xx     # PowerVR SGX 5XX series
#       --kernel-variant=android-gpu-adreno3xx  # Qualcomm Adreno 3XX series
#     If you don't choose one, the program will still run, but Memcheck
#     may report false errors after the program performs GPU-specific ioctls.
#
# Anyway: to run on the device:
#
/data/local/Inst/bin/valgrind [kernel variant args] [the usual args etc]


# Once you're up and running, a handy modify-V-rebuild-reinstall
# command line (on the host, of course) is
#
mq -j2 && mq -j2 install DESTDIR=`pwd`/Inst && adb push Inst /
#
# where 'mq' is an alias for 'make --quiet'.


# One common cause of runs failing at startup is the inability of
# Valgrind to find a suitable temporary directory.  On the device,
# there doesn't seem to be any one location which we always have
# permission to write to.  The instructions above use /sdcard.  If
# that doesn't work for you, and you're Valgrinding one specific
# application which is already installed, you could try using its
# temporary directory, in /data/data, for example
# /data/data/org.mozilla.firefox_beta.
#
# Using /system/bin/logcat on the device is helpful for diagnosing
# these kinds of problems.
