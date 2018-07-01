
#A small guide to compile openFrameworks armv7 with driver OpenGL ES / ES2 sunxi for H3 / Mali GPU on ARMBIAN OS, compatible with orangepi and bananapi.

tested on: orangepi one banana pi M2 orange pi pc

dependencies:

apt-get install libx11-dev libxext-dev xutils-dev libdrm-dev \ 
           x11proto-xf86dri-dev libxfixes-dev x11proto-dri2-dev xserver-xorg-dev \
           build-essential automake pkg-config libtool ca-certificates git cmake subversion


3dpart package:

git clone https://github.com/linux-sunxi/sunxi-mali
git clone https://github.com/robclark/libdri2
git clone https://github.com/linux-sunxi/libump
git clone https://github.com/ssvb/xf86-video-fbturbo
git clone https://github.com/ptitSeb/glshim


Install:

compile libdri2:

cd libdri2/
autoreconf -i
./configure --prefix=/usr
make
make install


compile libump:

cd libump/
autoreconf -i
./configure --prefix=/usr
make
make install


compile mali:

cd sunxi-mali/
git submodule init
git submodule update
git pull
wget http://pastebin.com/raw.php?i=hHKVQfrh -O ./include/GLES2/gl2.h
wget http://pastebin.com/raw.php?i=ShQXc6jy -O ./include/GLES2/gl2ext.h
make config ABI=armhf VERSION=r3p0
mkdir /usr/lib/mali
echo "/usr/lib/mali" > /etc/ld.so.conf.d/1-mali.conf
make -C include install
make -C lib/mali prefix=/usr libdir='$(prefix)/lib/mali/' install


compile fbturbo:

cd xf86-video-fbturbo
autoreconf -i
./configure --prefix=/usr
make
make install


compile glshim:

cd glshim
cmake .
make
cp lib/libGL.so.1 /usr/lib/ 


Assign permissions group video for /dev/ump and /dev/mali

vi /etc/udev/rules.d/50-mali.rules

add this line:

KERNEL=="mali", MODE="0660", GROUP="video"
KERNEL=="ump", MODE="0660", GROUP="video"


add swap if you work with 512MB of RAM:

dd if=/dev/zero of=/swapmem bs=1024 count=524288
chown root:root /swapmem
chmod 0600 /swapmem
mkswap /swapmem
swapon /swapmem
vi /etc/fstab

add line file /etc/fstab:

/swapmem none swap sw 0 0


compile poco: (for this step I recommend the use of apothecary)

wget https://pocoproject.org/releases/poco-1.7.7/poco-1.7.7-all.tar.gz
tar xvf poco-1.7.7-all.tar.gz
cd poco-1.7.7-all/
./configure --static --no-tests --no-samples \
--omit=CppUnit,CppUnit/WinTestRunner,Data/MySQL,Data/ODBC,PageCompiler,PageCompiler/File2Page,CppParser,PDF,PocoDoc,ProGen
make -j3


compile openFrameworks:

wget http://openframeworks.cc/versions/v0.9.8/of_v0.9.8_linuxarmv7l_release.tar.gz
tar xvf of_v0.9.8_linuxarmv7l_release.tar.gz
cd of_v0.9.8_linuxarmv7l_release/scripts/linux/debian/
./install_codecs.sh
./install_dependencies.sh
cp ~/poco-1.7.7-all/lib/Linux/armv7l/* ~/of_v0.9.8_linuxarmv7l_release/libs/poco/lib/linuxarmv7l/
cd ~/of_v0.9.8_linuxarmv7l_release/apps/myApps/emptyExample/
make


configure OF optional:

cp ~/of_v0.9.8_linuxarmv7l_release/libs/openFrameworksCompiled/project/linuxarmv7l/config.linuxarmv7l.default.mk \
~/of_v0.9.8_linuxarmv7l_release/libs/openFrameworksCompiled/project/linuxarmv7l/config.linuxarmv7l.h3.mk

and change CFLAGS -mtune=cortex-a8 in -mtune=cortex-a7

PLATFORM_CFLAGS += -mtune=cortex-a7

for compile example OF with variant H3:

make PLATFORM_VARIANT=h3


URL Utility:

    http://linux-sunxi.org/Xorg
    http://linux-sunxi.org/Mali_binary_driver
    http://linux-sunxi.org/KernelArguments
    https://docs.armbian.com

