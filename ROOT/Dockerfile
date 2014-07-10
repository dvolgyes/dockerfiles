# 
#  Dockerfile for ROOT Data Analysis Framework  
#  http://root.cern.ch/drupal/
#
#  The image is based on the latest official Ubuntu (14.04) and on the latest official ROOT release (6.00.02)
#
#

FROM ubuntu
MAINTAINER David Völgyes <david.volgyes@ieee.org>

# Setting environment variables
ENV TERM linux
ENV DEBIAN_FRONTEND noninteractive

# Fixing an Ubuntu related problem, see details here: https://github.com/dotcloud/docker/issues/1024
RUN dpkg-divert --local --rename /usr/bin/ischroot && ln -sf /bin/true /usr/bin/ischroot

RUN  apt-get update
RUN  apt-get install    -y --force-yes dialog debconf-utils
RUN  apt-get dist-upgrade -y --force-yes
RUN  apt-get install  -y --force-yes git gcc g++ cmake libfftw3-dev libgomp1 graphviz-dev libgsl0-dev freeglut3-dev libxpm-dev libpcre3-dev libxft-dev liblzma-dev python-dev libjpeg-dev libpng-dev libtiff-dev libxml2-dev sqlite libpythia8-dev libgif-dev gfortran 
RUN  apt-get clean

RUN mkdir -p /opt/src
RUN mkdir -p /opt/root
RUN cd /opt/src ; git clone http://root.cern.ch/git/root.git
RUN mv /opt/src/root /opt/src/root-src
RUN cd /opt/src/root-src ; git checkout -b v6-00-02
RUN cd /opt/root ; cmake -D CMAKE_INSTALL_PREFIX=/usr -D CMAKE_BUILD_TYPE=Release -D builtin_gsl=ON  -D python=ON -D soversion=ON   /opt/src/root-src/
RUN make -C /opt/root
RUN make install  -C /opt/root
