Build Qt 5.2.1
==============

[Original source: [http://qt-project.org/wiki/TIPandaBoard](http://qt-project.org/wiki/TIPandaBoard)]

Make sure we have our cross-compiler installed. 
Ubuntu provides one for host machine:
```
sudo apt-get g++-arm-linux-gnueabihf build-essential
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

Now configure (note the last 3 include arguments; these are required due to funky placement of the GBM/DRM headers in 
the TI repository):
```
./configure \
-v \
-verbose \
-device linux-pandaboard-g++ \
-nomake tests \
-nomake examples \
-make libs \
-make examples \
-make demos \
-device-option CROSS_COMPILE=arm-linux-gnueabihf- \
-prefix /opt/Qt-5.2.1-Pandaboard \
-sysroot [your_rootfs_path] \
-confirm-license \
-opensource \
-force-pkg-config 
```

If all goes well, you should be able to run
```
make && make install
```
to finish building Qt. 

If an important feature is missing (like OpenGL ES 2), run configure with -v for potentially helpful error messages. 
It should be possible to build OpenGL ES 2 along with the EGLFS, minimal EGL, and KMS QPA plugins.

Finally copy our new library on the target:
```
cp -R /opt/Qt-5.2.1-Pandaboard/mkspec /your_rootfs_path/usr/local/Qt-5.2.1-Pandaboard
```

and add Qt to the system path
* TODO...*
