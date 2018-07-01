#A small guide to compile openFrameworks armv7 with driver OpenGL ES / ES2 sunxi for H3 / Mali GPU on <a href="https://www.armbian.com/">ARMBIAN OS</a>, compatible with orangepi and bananapi.

tested on:
orangepi one
banana pi M2
orange pi pc

<b>dependencies:</b>
<pre><code>apt-get install libx11-dev libxext-dev xutils-dev libdrm-dev \ 
           x11proto-xf86dri-dev libxfixes-dev x11proto-dri2-dev xserver-xorg-dev \
           build-essential automake pkg-config libtool ca-certificates git cmake subversion
</code></pre>

<br>
<b>3dpart package:</b>
<pre><code>git clone https://github.com/linux-sunxi/sunxi-mali<br>
git clone https://github.com/robclark/libdri2<br>
git clone https://github.com/linux-sunxi/libump<br>
git clone https://github.com/ssvb/xf86-video-fbturbo<br>
git clone https://github.com/ptitSeb/glshim<br>
</code></pre>
<br>
<b>Install:</b><br>
<br>
<b>compile libdri2:</b>
<pre><code>cd libdri2/<br>
autoreconf -i<br>
./configure --prefix=/usr<br>
make<br>
make install<br>
</code></pre>
<br>
<b>compile libump:</b>
<pre><code>cd libump/<br>
autoreconf -i<br>
./configure --prefix=/usr<br>
make<br>
make install<br>
</code></pre>
<br>
<b>compile mali:</b>
<pre><code>cd sunxi-mali/<br>
git submodule init<br>
git submodule update<br>
git pull<br>
wget http://pastebin.com/raw.php?i=hHKVQfrh -O ./include/GLES2/gl2.h<br>
wget http://pastebin.com/raw.php?i=ShQXc6jy -O ./include/GLES2/gl2ext.h<br>
make config ABI=armhf VERSION=r3p0<br>
mkdir /usr/lib/mali<br>
echo "/usr/lib/mali" > /etc/ld.so.conf.d/1-mali.conf<br>
make -C include install<br>
make -C lib/mali prefix=/usr libdir='$(prefix)/lib/mali/' install<br>
</code></pre>
<br>
<b>compile fbturbo:</b>
<pre><code>cd xf86-video-fbturbo<br>
autoreconf -i<br>
./configure --prefix=/usr<br>
make<br>
make install<br>
</code></pre>
<br>
<b>compile glshim:</b>
<pre><code>cd glshim<br>
cmake .<br>
make<br>
cp lib/libGL.so.1 /usr/lib/ <br>
</code></pre>
<br>
<b>Assign permissions group video for /dev/ump and /dev/mali</b>
<pre><code>vi /etc/udev/rules.d/50-mali.rules</code></pre>
<b>add this line:</b>
<pre><code>KERNEL=="mali", MODE="0660", GROUP="video"
KERNEL=="ump", MODE="0660", GROUP="video"</code></pre>
<br>
<b>add swap if you work with 512MB of RAM:</b>
<pre><code>dd if=/dev/zero of=/swapmem bs=1024 count=524288<br>
chown root:root /swapmem<br>
chmod 0600 /swapmem<br>
mkswap /swapmem<br>
swapon /swapmem<br>
vi /etc/fstab<br>
</code></pre>
<b>add line file /etc/fstab:</b>
<pre><code>/swapmem none swap sw 0 0</code></pre>
<br>
<b>compile poco: (for this step I recommend the use of apothecary)</b>
<pre><code>wget https://pocoproject.org/releases/poco-1.7.7/poco-1.7.7-all.tar.gz
tar xvf poco-1.7.7-all.tar.gz
cd poco-1.7.7-all/
./configure --static --no-tests --no-samples \
--omit=CppUnit,CppUnit/WinTestRunner,Data/MySQL,Data/ODBC,PageCompiler,PageCompiler/File2Page,CppParser,PDF,PocoDoc,ProGen
make -j3
</code></pre>
<br>
<b>compile openFrameworks:</b>
<pre><code>wget http://openframeworks.cc/versions/v0.9.8/of_v0.9.8_linuxarmv7l_release.tar.gz
tar xvf of_v0.9.8_linuxarmv7l_release.tar.gz
cd of_v0.9.8_linuxarmv7l_release/scripts/linux/debian/
./install_codecs.sh
./install_dependencies.sh
cp ~/poco-1.7.7-all/lib/Linux/armv7l/* ~/of_v0.9.8_linuxarmv7l_release/libs/poco/lib/linuxarmv7l/
cd ~/of_v0.9.8_linuxarmv7l_release/apps/myApps/emptyExample/
make
</code></pre>
<br>
<b>configure OF optional:</b>
<pre><code>cp ~/of_v0.9.8_linuxarmv7l_release/libs/openFrameworksCompiled/project/linuxarmv7l/config.linuxarmv7l.default.mk \
~/of_v0.9.8_linuxarmv7l_release/libs/openFrameworksCompiled/project/linuxarmv7l/config.linuxarmv7l.h3.mk</code></pre>
<b>and change CFLAGS -mtune=cortex-a8 in -mtune=cortex-a7</b>
<pre><code>PLATFORM_CFLAGS += -mtune=cortex-a7</code></pre>
<b>for compile example OF with variant H3:</b>
<pre><code>make PLATFORM_VARIANT=h3</code></pre>
<br>
<b>URL Utility:</b><br>
<ul>
<li>http://linux-sunxi.org/Xorg</li>
<li>http://linux-sunxi.org/Mali_binary_driver</li>
<li>http://linux-sunxi.org/KernelArguments</li>
<li>https://docs.armbian.com</li>
</ul>
