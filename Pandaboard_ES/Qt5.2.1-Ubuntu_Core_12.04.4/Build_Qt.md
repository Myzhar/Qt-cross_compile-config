Build Qt 5.2.1
==============

Make sure you have your native and cross-compilers installed. Ubuntu provides one (so does Linaro):
```
sudo apt-get g++-arm-linux-gnueabihf build-essential
```
You will want to mount that rootfs from the previous section to make life easier when building Qt. From above:
```
mkdir rootfs
sudo mount -o loop,offset=41126400 disk.img rootfs
```

Download Pandaboard mkspec and copy it in the right folder:
*TODO creare un file di archivio con i file necessari e renderlo disponibile*

Now configure (note the last 3 include arguments; these are required due to funky placement of the GBM/DRM headers in 
the TI repository):
```
./configure \
-device linux-pandaboard-g++ \
-nomake tests -nomake examples \
-prefix /opt/Qt-5.2.1-Pandaboard \
-sysroot /your_rootfs_path
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
