Build Qt 5.2.1
==============

[Original source: [http://qt-project.org/wiki/TIPandaBoard](http://qt-project.org/wiki/TIPandaBoard)]

Make sure we have our cross-compiler installed. 
Ubuntu provides one for host machine:
```
sudo apt-get g++-arm-linux-gnueabihf build-essential
```

[*Try to use* **gcc-Linaro-arm-linux-gnueabihf-4.8-2013.09_linux**]

If the host runs Linux 64 bit install 32bit libs:
```
sudo apt-get install ia32-libs
```

Mount the rootfs from "Config_Ubuntu" guide to make life easier when building Qt. From above:
```
sudo mount -o loop,offset=41126400 disk.img rootfs
```

Download Pandaboard mkspec and copy it in the right folder:

*TODO creare un file di archivio con i file necessari e renderlo disponibile*
```
cp -R linux-pandaboard-g++ [path_to_qt]/qtbase/mkspecs/devices/
```

Fix the relative paths using the *fixQualifiedLibraryPaths* available [here](https://gitorious.org/cross-compile-tools/cross-compile-tools/source/98c51c5939d91884b096dd2fbee859803fd34fef:fixQualifiedLibraryPaths):
```
git clone https://git.gitorious.org/cross-compile-tools/cross-compile-tools.git
sudo ./fixQualifiedLibraryPaths [your_rootfs_path] [your_crosscompiler_path]
```

Now configure (*note the use of the 3rd party libraries, instead of system ones to avoid any conflict of version *):
```
./configure \
-v \
-verbose \
-device linux-pandaboard-g++ \
-nomake tests \
-make libs \
-make examples \
-make demos \
-device-option CROSS_COMPILE=arm-linux-gnueabihf- \
-prefix /opt/Qt-5.2.1-Pandaboard \
-sysroot [your_rootfs_path] \
-confirm-license \
-opensource \
-force-pkg-config \
-qt-zlib \
-qt-libjpeg \
-qt-libpng \
-qt-xcb \
-qt-xkbcommon \
-qt-freetype \
-qt-pcre \
-qt-harfbuzz \
-qpa eglfs
```

If all goes well, you should be able to run
```
make -jX && make install
```
to finish building Qt (*replace* **X** *with the number of core of the host machine to speedup compiling process*).

If an important feature is missing (like OpenGL ES 2), run configure with -v for potentially helpful error messages. 
It should be possible to build OpenGL ES 2 along with the EGLFS, minimal EGL, and KMS QPA plugins.

Finally copy our new library on the target:
```
cp -R /opt/Qt-5.2.1-Pandaboard/mkspec /your_rootfs_path/usr/local/Qt-5.2.1-Pandaboard
```

and add Qt to the system path
* TODO...*
