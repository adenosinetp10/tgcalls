# Build instruction for macOS

XCode Command Line Tools and python3 need to installed.

```shell script
MAKE_THREADS_CNT=-j8
MACOSX_DEPLOYMENT_TARGET=10.12
UNGUARDED="-Werror=unguarded-availability-new"
MIN_VER="-mmacosx-version-min=10.12"

ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew install automake cmake git libvpx ninja opus yasm libtool

git clone --recursive https://github.com/MarshalX/tgcalls

mkdir -p Libraries/macos
cd Libraries/macos

git clone https://github.com/desktop-app/patches.git
cd patches
git checkout e052c49
cd ..

git clone -b v4.0.1-rc2 https://github.com/mozilla/mozjpeg.git
cd mozjpeg
cmake -B build . \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/usr/local/macos \
    -DCMAKE_OSX_DEPLOYMENT_TARGET:STRING=10.12 \
    -DWITH_JPEG8=ON \
    -DPNG_SUPPORTED=OFF
cmake --build build $MAKE_THREADS_CNT
sudo cmake --install build
cd ..

git clone https://github.com/openssl/openssl openssl_1_1_1
cd openssl_1_1_1
git checkout OpenSSL_1_1_1-stable
./Configure --prefix=/usr/local/macos no-tests darwin64-x86_64-cc -static $MIN_VER
make build_libs $MAKE_THREADS_CNT
cd ..

git clone https://github.com/xiph/opus
cd opus
git checkout v1.3
./autogen.sh
CFLAGS="$MIN_VER $UNGUARDED" CPPFLAGS="$MIN_VER $UNGUARDED" LDFLAGS="$MIN_VER" ./configure --prefix=/usr/local/macos
make $MAKE_THREADS_CNT
sudo make install
cd ..

git clone https://github.com/FFmpeg/FFmpeg.git ffmpeg
cd ffmpeg
git checkout release/4.2
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig:/usr/lib/pkgconfig:/usr/X11/lib/pkgconfig
cp ../patches/macos_yasm_wrap.sh ./

./configure --prefix=/usr/local/macos \
--extra-cflags="$MIN_VER $UNGUARDED" \
--extra-cxxflags="$MIN_VER $UNGUARDED" \
--extra-ldflags="$MIN_VER" \
--x86asmexe=`pwd`/macos_yasm_wrap.sh \
--enable-protocol=file \
--enable-libopus \
--disable-programs \
--disable-doc \
--disable-network \
--disable-everything \
--enable-hwaccel=h264_videotoolbox \
--enable-hwaccel=hevc_videotoolbox \
--enable-hwaccel=mpeg1_videotoolbox \
--enable-hwaccel=mpeg2_videotoolbox \
--enable-hwaccel=mpeg4_videotoolbox \
--enable-decoder=aac \
--enable-decoder=aac_at \
--enable-decoder=aac_fixed \
--enable-decoder=aac_latm \
--enable-decoder=aasc \
--enable-decoder=alac \
--enable-decoder=alac_at \
--enable-decoder=flac \
--enable-decoder=gif \
--enable-decoder=h264 \
--enable-decoder=hevc \
--enable-decoder=mp1 \
--enable-decoder=mp1float \
--enable-decoder=mp2 \
--enable-decoder=mp2float \
--enable-decoder=mp3 \
--enable-decoder=mp3adu \
--enable-decoder=mp3adufloat \
--enable-decoder=mp3float \
--enable-decoder=mp3on4 \
--enable-decoder=mp3on4float \
--enable-decoder=mpeg4 \
--enable-decoder=msmpeg4v2 \
--enable-decoder=msmpeg4v3 \
--enable-decoder=opus \
--enable-decoder=pcm_alaw \
--enable-decoder=pcm_alaw_at \
--enable-decoder=pcm_f32be \
--enable-decoder=pcm_f32le \
--enable-decoder=pcm_f64be \
--enable-decoder=pcm_f64le \
--enable-decoder=pcm_lxf \
--enable-decoder=pcm_mulaw \
--enable-decoder=pcm_mulaw_at \
--enable-decoder=pcm_s16be \
--enable-decoder=pcm_s16be_planar \
--enable-decoder=pcm_s16le \
--enable-decoder=pcm_s16le_planar \
--enable-decoder=pcm_s24be \
--enable-decoder=pcm_s24daud \
--enable-decoder=pcm_s24le \
--enable-decoder=pcm_s24le_planar \
--enable-decoder=pcm_s32be \
--enable-decoder=pcm_s32le \
--enable-decoder=pcm_s32le_planar \
--enable-decoder=pcm_s64be \
--enable-decoder=pcm_s64le \
--enable-decoder=pcm_s8 \
--enable-decoder=pcm_s8_planar \
--enable-decoder=pcm_u16be \
--enable-decoder=pcm_u16le \
--enable-decoder=pcm_u24be \
--enable-decoder=pcm_u24le \
--enable-decoder=pcm_u32be \
--enable-decoder=pcm_u32le \
--enable-decoder=pcm_u8 \
--enable-decoder=pcm_zork \
--enable-decoder=vorbis \
--enable-decoder=wavpack \
--enable-decoder=wmalossless \
--enable-decoder=wmapro \
--enable-decoder=wmav1 \
--enable-decoder=wmav2 \
--enable-decoder=wmavoice \
--enable-encoder=libopus \
--enable-parser=aac \
--enable-parser=aac_latm \
--enable-parser=flac \
--enable-parser=h264 \
--enable-parser=hevc \
--enable-parser=mpeg4video \
--enable-parser=mpegaudio \
--enable-parser=opus \
--enable-parser=vorbis \
--enable-demuxer=aac \
--enable-demuxer=flac \
--enable-demuxer=gif \
--enable-demuxer=h264 \
--enable-demuxer=hevc \
--enable-demuxer=m4v \
--enable-demuxer=mov \
--enable-demuxer=mp3 \
--enable-demuxer=ogg \
--enable-demuxer=wav \
--enable-muxer=ogg \
--enable-muxer=opus

make $MAKE_THREADS_CNT
sudo make install
cd ..

cp -R ../../tgcalls/tgcalls/third_party/webrtc tg_owt
cd tg_owt
mkdir -p out/Debug
cd out/Debug
cmake -G Ninja \
    -DCMAKE_BUILD_TYPE=Debug \
    -DTG_OWT_SPECIAL_TARGET=mac \
    -DTG_OWT_LIBJPEG_INCLUDE_PATH=/usr/local/macos/include \
    -DTG_OWT_OPENSSL_INCLUDE_PATH=`pwd`/../../../openssl_1_1_1/include \
    -DTG_OWT_OPUS_INCLUDE_PATH=/usr/local/macos/include/opus \
    -DTG_OWT_FFMPEG_INCLUDE_PATH=/usr/local/macos/include ../..
ninja
cd ..
mkdir Release
cd Release
cmake -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DTG_OWT_SPECIAL_TARGET=mac \
    -DTG_OWT_LIBJPEG_INCLUDE_PATH=/usr/local/macos/include \
    -DTG_OWT_OPENSSL_INCLUDE_PATH=`pwd`/../../../openssl_1_1_1/include \
    -DTG_OWT_OPUS_INCLUDE_PATH=/usr/local/macos/include/opus \
    -DTG_OWT_FFMPEG_INCLUDE_PATH=/usr/local/macos/include ../..
ninja

cd ../../../../../tgcalls/

python3 setup.py build
```