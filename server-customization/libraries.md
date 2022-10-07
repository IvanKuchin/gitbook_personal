# Libraries

### Crypto

Libcrypto required to calculate SHA512. Has mostly used in password generation and in single function in utilities.h

To function properly required standard libcrypto installation.

### libmaxminddb

```
sudo apt install libmaxminddb-dev
```

### libwkhtmltox

Library allows to convert HTML to PDF. [Installation instructions](https://gist.github.com/srmds/2507aa3bcdb464085413c650fe42e31d)

```
wget https://github.com/wkhtmltopdf/wkhtmltopdf/releases/download/0.12.5/wkhtmltox_0.12.5-1.trusty_amd64.deb
sudo dpkg -i  wkhtmltox_0.12.5-1.trusty_amd64.deb 
sudo apt -f install
```

compilation

```
g++ -c -I /usr/local/include/wkhtmltox ./pdf_c_api.c
g++ -L /usr/local/lib pdf_c_api.o -lwkhtmltox
```

### ImageMagick++

```
sudo apt install graphicsmagick-libmagick-dev-compat libgraphicsmagick++1-dev libwebp-dev
```

| graphicsmagick-libmagick-dev-compat | For Magick++-config   |
| ----------------------------------- | --------------------- |
| libgraphicsmagick++1-dev            | Magick library itself |
| libwebp-dev                         | Missed dependencies   |

### libwebsocket

Options list can be found in CMakeList.txt

```
cmake – LH ..
```

```
cd /usr/src
tar –zxf <>
cd <xxx>
mkdir build
cd build
cmake -DLWS_WITHOUT_DAEMONIZE:BOOL=FALSE ..
make
sudo make install
```

```
# with DEBUG info
cmake -DCMAKE_BUILD_TYPE=DEBUG ..

# installation path
cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr ..
```

#### Upgrade from 2.2 to 4.1

See comment [here](https://github.com/warmcat/libwebsockets/issues/2219).

Broadly it will work the same, although 2.2 is very old. V2.2 is already after the big rationalization of api names to `lws_` that made the most churn.

The default loop became smarter about scheduling, and more like an event library. You should use the lws\_sul apis

[https://libwebsockets.org/git/libwebsockets/tree/READMEs/README.lws\_sul.md?h=main](https://libwebsockets.org/git/libwebsockets/tree/READMEs/README.lws\_sul.md?h=main)

if you want to schedule arbitrary things (in the own callback, outside of any protocol) to happen at arbitrary times.

### ffmpeg

“Universe” repository includes ffmpeg-2.8.11. Which is ancient and having issues with transcoding to mp4 (ffmpeg –i test.mov test.mp4 will fail). Last stable ffmpeg (3.2.4) required. Build it from source is painful because of lot dependencies. Another option is to use PPA (Personal Package Archive).

FFMPEG-3 PPA ([https://launchpad.net/\~jonathonf/+archive/ubuntu/ffmpeg-4](https://launchpad.net/\~jonathonf/+archive/ubuntu/ffmpeg-4)).

Add PPA to apt repositories and install ffmpeg and dependencies (install/uninstall process described [here](http://tipsonubuntu.com/2016/11/02/install-ffmpeg-3-2-via-ppa-ubuntu-16-04/)):

```
sudo add-apt-repository ppa:jonathonf/ffmpeg-4
sudo apt update
apt show ffmpeg (shows last version)
    OR  
apt-cache show ffmpeg (shows all available versions)
apt install ffmpeg
```

To uninstall FFmpeg 3.2 and revert to the stock version (2.8.8) in Ubuntu 16.04, simply purge the PPA via command:

```
sudo apt install ppa-purge
sudo ppa-purge ppa:jonathonf/ffmpeg-4
sudo apt autoremove
```

Check that transcoding to .mp4 and .webm supported

```
ffmpeg –i test.mov test.mp4
ffmpeg –i test.mov test.webm
```

#### Performance

Transcoding 30 sec MOV with initial size 51MB

![](../.gitbook/assets/ffmper\_performance.jpg)
