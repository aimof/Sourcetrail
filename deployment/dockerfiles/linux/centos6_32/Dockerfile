FROM toopher/centos-i386:centos6

MAINTAINER "Andreas Stallinger" <astallinger@coati.io>

WORKDIR /opt

RUN yum -y install wget && \
wget https://copr.fedorainfracloud.org/coprs/mlampe/devtoolset-4.1/repo/\
epel-6/mlampe-devtoolset-4.1-epel-6.repo -O /etc/yum.repos.d/mlampe-devtoolset.repo && \
wget https://copr.fedorainfracloud.org/coprs/tmz/git/repo/epel-6/tmz-git-epel-6.repo \
 -O /etc/yum.repos.d/git.repo && \
yum -y install epel-release && \
wget https://centos6.iuscommunity.org/ius-release.rpm && \
rpm -Uvh ius-release*.rpm && \
yum -y update && \
yum -y install devtoolset-4-gcc devtoolset-4-gcc-c++ devtoolset-4-binutils \
	wget tar bzip2 git libtool which fuse fuse-devel libpng-devel automake \
	glibc-headers libstdc++-devel gcc-c++ freetype-devel fontconfig-devel \
	ibxml2-devel libstdc++-devel libXrender-devel patch xcb-util-keysyms-devel \
	libXi-devel libudev-devel openssl-devel sqlite-devel \
	gperftools gperf libicu-devel boost-devel \
	libxslt-devel docbook-style-xsl.noarch python27 \
	xz mesa-libEGL-devel mesa-libGL-devel glib-devel \
	java-1.8.0-openjdk-devel ccache re2c ImageMagick vim lftp && \
yum clean all

# CMake and Maven
RUN wget https://cmake.org/files/v3.8/cmake-3.8.2.tar.gz && \
tar xf cmake-3.8.2.tar.gz && \
cd cmake-3.8.2 && ./bootstrap && \
make && make install && \
cd .. && rm cmake-3.8.2.tar.gz && \
rm -rf cmake-3.8.2 && \
wget https://archive.apache.org/dist/maven/maven-3/3.5.0/binaries/apache-maven-3.5.0-bin.tar.gz && \
tar xzf apache-maven-3.5.0-bin.tar.gz && \
ln -s apache-maven-3.5.0 maven && \
rm apache-maven-3.5.0-bin.tar.gz

ENV M2_HOME=/opt/maven 
ENV PATH=${M2_HOME}/bin:${PATH}

# Install Qt
ARG QVERSION_SHORT=5.9
ARG QTVERSION=5.9.1

RUN mkdir -p /qt && cd /qt && \
wget http://download.qt.io/archive/qt/${QVERSION_SHORT}/${QTVERSION}/\
single/qt-everywhere-opensource-src-${QTVERSION}.tar.xz && \
tar xvf qt-everywhere-opensource-src-${QTVERSION}.tar.xz && \
ln -sf /opt/rh/devtoolset-4/root/usr/bin/g++ /usr/bin/g++ && \
ln -sf /opt/rh/devtoolset-4/root/usr/bin/c++ /usr/bin/c++ && \
cd /qt/qt-everywhere-opensource-src-${QTVERSION} && \
./configure -v -prefix /opt/qt${QTVERSION} -skip qtgamepad -platform linux-g++ -qt-pcre \
-qt-xcb -qt-xkbcommon -xkb-config-root /usr/share/X11/xkb \
-no-pch -confirm-license -opensource && \
make -j 8 || make -j 1 install; make -j8 install && rm -Rf /qt

## set env
ENV CXX=g++ \
CC=gcc \
DEVTOOLSET=/opt/rh/devtoolset-4/root/usr/

## Install llvm
ARG LLVM_VERSION=5.0.0
RUN alias python=/usr/bin/python2.7 && mkdir -p /llvm && cd /llvm && \
wget http://llvm.org/releases/${LLVM_VERSION}/llvm-${LLVM_VERSION}.src.tar.xz && \
tar xf llvm-${LLVM_VERSION}.src.tar.xz && \
cd llvm-${LLVM_VERSION}.src && pwd && \
cd tools && pwd && \
wget http://llvm.org/releases/${LLVM_VERSION}/cfe-${LLVM_VERSION}.src.tar.xz && \
tar xf cfe-${LLVM_VERSION}.src.tar.xz && \
cd /llvm/llvm-${LLVM_VERSION}.src/projects && \
wget http://llvm.org/releases/${LLVM_VERSION}/compiler-rt-${LLVM_VERSION}.src.tar.xz && \
tar xf compiler-rt-${LLVM_VERSION}.src.tar.xz && \
#. /opt/rh/python27/enable && \
. /opt/rh/devtoolset-4/enable && \
python --version && \
cd /llvm/llvm-${LLVM_VERSION}.src && mkdir -p build && cd build && \
cmake .. -DCMAKE_INSTALL_PREFIX=/opt/llvm/ \
-DCMAKE_BUILD_TYPE=Release \
-DLLVM_ENABLE_RTTI=ON \
&& make -j8 install && rm -Rf /llvm
#COPY scripts/install-llvm.sh /opt/
#RUN ./install-llvm.sh

## Install botan
RUN git clone https://github.com/randombit/botan.git /botan && \
cd /botan && \
git checkout 2.1.0 && \
./configure.py --disable-shared --cpu=x86_32 --prefix=/opt/botan && \
make -j8 && make install && \
rm -rf /botan

## Boost
ARG BOOST_MAJOR=1
ARG BOOST_MINOR=64
ARG BOOST_PATCH=0
ENV BOOST_VERSION=${BOOST_MAJOR}.${BOOST_MINOR}.${BOOST_PATCH} \
BOOST_VERSION_UNDERSCORE=${BOOST_MAJOR}_${BOOST_MINOR}_${BOOST_PATCH}

RUN wget http://downloads.sourceforge.net/project/boost/boost/${BOOST_VERSION}/boost_${BOOST_VERSION_UNDERSCORE}.tar.gz && \
tar -xzf boost_${BOOST_VERSION_UNDERSCORE}.tar.gz && cd boost_${BOOST_VERSION_UNDERSCORE} && \
./bootstrap.sh --with-libraries=filesystem,program_options,system,date_time --prefix=/opt/boost && \
./b2 install --link=static --variant=release --threading=multi --runtime-link=static --cxxflags=-fPIC && \
cd .. && rm boost_${BOOST_VERSION_UNDERSCORE}.tar.gz && rm boost_${BOOST_VERSION_UNDERSCORE} -r

## Ninja
RUN git clone https://github.com/martine/ninja.git && \
cd ninja && \
git checkout release && \
./configure.py --bootstrap && \
mv ninja /usr/bin/ && \
cd .. && rm -rf ninja

## CxxTest
RUN git clone https://github.com/CxxTest/cxxtest.git && \
cd cxxtest && \
git checkout 4.4

#add user
RUN useradd builder

# Make sure the above SCLs are already enabled
#ENTRYPOINT ["/bin/bash", "-c", "source", "/opt/rh/devtoolset-4/enable"]
#CMD ["/bin/bash", "-c", "source", "/opt/rh/devtoolset-4/enable"]

RUN GCC_VERSION=$(g++ -dumpversion) && \
ln -s /opt/rh/devtoolset-4/root/usr/include/c++/${GCC_VERSION} /usr/include/c++/${GCC_VERSION} && \
ln -s /opt/rh/devtoolset-4/root/usr/lib/gcc/i686-redhat-linux/${GCC_VERSION} \
/usr/lib/gcc/i686-redhat-linux/${GCC_VERSION} && \
echo "source /opt/rh/devtoolset-4/enable" >> /etc/bashrc
## set env
ENV CXX=/opt/rh/devtoolset-4/root/usr/bin/g++ \
CC=/opt/rh/devtoolset-4/root/usr/bin/gcc \
BOTAN_DIR=/opt/botan \
CXX_TEST_DIR=/opt/cxxtest \
JAVA_HOME=/usr/lib/jvm/java-openjdk \
LLVM_DIR=/opt/llvm \
BOOST_DIR=/opt/boost \
QT_DIR=/opt/qt${QTVERSION}

WORKDIR /home/builder
USER builder
