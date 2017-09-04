# Latest imagemagick on google cloud functions

We needed latest imagemagick features for image processing on google cloud functions.

### Build imagemagick

We can build imagemagick starting from a simple ubuntu docker image, run the below script and take `/tmp/im` from the docker container, remove `share` and `include` folders and copy everything else into your google cloud functions

Note:
 - current script just installs png/jpeg support, nothing else
 - be sure to export the env variables needed to make imagemagick aware of the library paths (seems static version doesn't work perfectly and still needs jpeg and other libraries in `lib` folder)

```
# On ubuntu docker
export BUILD_PATH=/tmp
export OUT_PATH=/tmp/im
export PKG_CONFIG_PATH=$OUT_PATH/lib/pkgconfig
export PATH=$OUT_PATH/bin:$PATH
export LIBRARY_PATH=$OUT_PATH/lib
export LD_LIBRARY_PATH=$OUT_PATH/lib
export CPATH=$OUT_PATH/include

# Install build dependencies
apt-get update && apt-get install -y build-essential curl pkg-config

# Remove out path if already exists
rm -Rf $OUT_PATH

# Build path
cd $BUILD_PATH

###########
# libjpeg #
###########

# Download jpeg dependency
curl http://www.ijg.org/files/jpegsrc.v9b.tar.gz -o jpeg.tar.gz
# Unzip
tar -xvf jpeg.tar.gz
# Get into folder
cd jpeg-9b
# Configure
./configure --prefix $OUT_PATH
# Make
make
# Install
make install

# Build path
cd $BUILD_PATH

#####################
# zlib (for libpng) #
#####################

# Download dependency
curl https://zlib.net/zlib-1.2.11.tar.gz -o zlib.tar.gz
# Unzip
tar -xvf zlib.tar.gz
# Get into folder
cd zlib-1.2.11
# Configure
./configure --prefix $OUT_PATH
# Make
make
# Install
make install

# Build path
cd $BUILD_PATH

##########
# libpng #
##########

# Download dependency
curl ftp://ftp-osl.osuosl.org/pub/libpng/src/libpng16/libpng-1.6.32.tar.gz -o png.tar.gz
# Unzip
tar -xvf png.tar.gz
# Get into folder
cd libpng-1.6.32
# Configure
./configure --prefix $OUT_PATH --enable-shared
# Make
make
# Install
make install

# Build path
cd $BUILD_PATH

###############
# Imagemagick #
###############

# Download dependency
curl https://www.imagemagick.org/download/ImageMagick.tar.gz -o imagemagick.tar.gz
# Unzip
tar -xvf imagemagick.tar.gz
# Get into folder
cd ImageMagick-7.0.7-0
# Configure
./configure --prefix $OUT_PATH --without-perl --without-magick-plus-plus --with-quantum-depth=8 --disable-openmp  --enable-shared=no --enable-static=yes --enable-zero-configuration
# Make
make
# Install
make install
```
