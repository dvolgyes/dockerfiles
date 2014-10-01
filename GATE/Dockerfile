# 
#  Dockerfile for GATE 7.0  (with Geant4 9.6, LMF 3.0, Root 5.34, ECAT7)
#  http://www.opengatecollaboration.org/
#
#  The image is directly compiled from the source codes.
#
#  

FROM ubuntu:14.04
MAINTAINER David Völgyes <david.volgyes AT ieee org>

#
# Setting environment variables and download necessary tools:
#

ENV TERM linux
ENV DEBIAN_FRONTEND noninteractive

RUN   apt-get update
RUN   apt-get install      -y --force-yes   dialog debconf-utils wget
RUN   apt-get dist-upgrade -y --force-yes

# Sources
ADD http://geant4.web.cern.ch/geant4/support/source/geant4.9.6.p03.tar.gz   /opt/geant-src/
ADD http://www.opengatecollaboration.org/sites/opengatecollaboration.org/files/gate_v7.0.tar_.gz /opt/
ADD http://www.opengatecollaboration.org/sites/opengatecollaboration.org/files/lmf_v3_0_tar_gz_42819.gz /opt/
ADD http://www.opengatecollaboration.org/sites/opengatecollaboration.org/files/ecat.tar.gz /opt/ecat/

# Unfortunately, Docker does not support ftp protocol in its ADD command, so let's use wget
RUN cd /opt/ && wget ftp://root.cern.ch/root/root_v5.34.21.source.tar.gz

# Tools

RUN   apt-get install -y --force-yes   unzip gcc-4.7 g++-4.7 g++-4.8 gcc-4.8 make cmake-curses-gui gzip tar mc 
RUN   apt-get install -y --force-yes wget dpkg-dev build-essential autoconf automake tcl tk g++ libglu1-mesa-dev libxt-dev libxmu-dev gfortran libxaw7-dev libX11-dev libxft-dev libxpm-dev libxt-dev freeglut3-dev x11proto-print-dev po-debconf libusb-dev libboost-dev libtool libc6-dev graphviz graphviz-dev libxext-dev libpcre3-dev libglew1.5-dev libfftw3-dev libftgl-dev graphviz-dev libgsl0-dev libkrb5-dev libssl-dev libxml2-dev libldap2-dev libavahi-compat-libdnssd-dev libncurses5-dev libglu1-mesa-dev libcfitsio3-dev libmotif4 libmotif-dev libxml2 libxml2-dev libqt4-opengl libqt4-opengl-dev libgl1-mesa-dev libglw1-mesa-dev libxpm4 libxerces-c3-dev libqt4-core qt4-qmake libqt4-dev libgtkgl2.0-dev libgtkglarea-cil-dev liblablgl-ocaml-dev liblablgl-ocaml libxerces-c-dev  libxmltooling-dev happycoders-libsocket-dev  libvtk5-dev libfftw3-3 libxt-dev libfftw3-dev 
RUN   cd /opt && wget http://mirrors.kernel.org/ubuntu/pool/universe/g/glui/libglui-dev_2.36-4ubuntu1_amd64.deb 
RUN   cd /opt && wget http://mirrors.kernel.org/ubuntu/pool/universe/g/glui/libglui2c2_2.36-4ubuntu1_amd64.deb 
RUN   dpkg -i /opt/*.deb
RUN   rm /opt/*.deb
RUN   apt-get install -f
RUN   apt-get clean


RUN    update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.7 47
RUN    update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.7 47

RUN    update-alternatives --install /usr/bin/cc cc /usr/bin/gcc-4.7 47
RUN    update-alternatives --install /usr/bin/c++ c++ /usr/bin/g++-4.7 47

RUN    update-alternatives --set gcc /usr/bin/gcc-4.7
RUN    update-alternatives --set g++ /usr/bin/g++-4.7
RUN    update-alternatives --set cc /usr/bin/gcc-4.7
RUN    update-alternatives --set c++ /usr/bin/g++-4.7

# ROOT
RUN tar -C /opt -xzvf /opt/root_v5.34.21.source.tar.gz
RUN mv /opt/root /opt/root-5.34.21

RUN cd /opt/root-5.34.21 && ./configure linuxx8664gcc --prefix=/usr
RUN make -C /opt/root-5.34.21 -j4
RUN make -C /opt/root-5.34.21 install
RUN echo "/usr/lib/root" >/etc/ld.so.conf.d/root.conf
RUN ldconfig

# Extract Geant4
RUN cd /opt/geant-src && tar -xvf geant4.9.6.p03.tar.gz

# Patch CLHEP bug
RUN sed -i "214i if(0) // disable this part, because it has a bug." /opt/geant-src/geant4.9.6.p03/source/externals/clhep/src/MTwistEngine.cc

#Configure, compile and install Geant4
RUN mkdir /opt/geant-build
RUN cd /opt/geant-build && cmake -DGEANT4_INSTALL_DATA=ON -DCMAKE_INSTALL_PREFIX=/usr /opt/geant-src/geant4.9.6.p03
RUN make -C /opt/geant-build -j4
RUN make -C /opt/geant-build install

# Extract Gate v7, LMF and ECAT sources
RUN cd /opt && tar -xvf gate_v7.0.tar_.gz 
RUN cd /opt && tar -xvf lmf_v3_0_tar_gz_42819.gz
RUN cd /opt/ecat && tar -xvf ecat.tar.gz

# LMF build and install
RUN cd /opt/lmf_v3.0 && sh configure
RUN cd /opt/lmf_v3.0 && make

RUN cp /opt/lmf_v3.0/includes/* /usr/local/include
RUN echo "/opt/lmf_v3.0/lib/" >/etc/ld.so.conf.d/lmf.conf

# ECAT build and install
RUN cp /opt/ecat/Makefile.unix /opt/ecat/Makefile
RUN make -C /opt/ecat
RUN cp /opt/ecat/libecat.a /usr/local/lib
RUN mkdir /opt/ecat/include
RUN cp /opt/ecat/*.h /opt/ecat/include
RUN ldconfig

# Do NOT use ECAT utils! It's full with bugs!
RUN rm -R /opt/ecat/utils

# GATE 
RUN mkdir /opt/gate
RUN cd /opt/gate && cmake -DGATE_USE_ECAT7=ON -DGATE_USE_LMF=ON -DECAT7_HOME=/opt/ecat -DLMF_HOME=/opt/lmf_v3.0 -DGATE_DOWNLOAD_BENCHMARKS_DATA=OFF -DGATE_DOWNLOAD_EXAMPLES_DATA=OFF   ../gate_v7.0
RUN make -C /opt/gate -j4
RUN make -C /opt/gate install
RUN ldconfig

ENV DEBIAN_FRONTEND dialog
