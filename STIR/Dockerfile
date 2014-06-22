# 
#  Dockerfile for STIR (Software for Tomographic Image Reconstruction)
#
#  Compile it that way: docker build -t stir .
#

# Using latest official Ubuntu image
FROM ubuntu

#
# Setting environment variables and download necessary tools:
#
ENV TERM linux
ENV DEBIAN_FRONTEND noninteractive

RUN   apt-get update
RUN   apt-get install      -y --force-yes dialog debconf-utils
RUN   apt-get dist-upgrade -y --force-yes
RUN   apt-get install      -y  --force-yes  unzip gcc g++ make  cmake-curses-gui libncurses-dev libx11-dev libboost-dev tcsh python python-dev mpi-default-dev mpi-default-bin libinsighttoolkit4-dev libtiff5-dev libgomp1 libgdcm2-dev gdb swig libpng12-dev xutils-dev libc6-dev libsocket++-dev tar gzip 
RUN   apt-get clean

#
# Adding STIR related files. 
# The STIR.zip and the recon_test_pack.zip must be in the same directory where Dockerfile is!
#

ADD  ./STIR.zip /opt/
ADD  ./recon_test_pack.zip /opt/
ADD  http://www.opengatecollaboration.org/sites/opengatecollaboration.org/files/ecat.tar.gz /opt/ecat/
RUN  mkdir -p /opt/STIR-bin/Release
RUN  cd /opt/ecat && tar xvzf ecat.tar.gz
RUN  unzip /opt/STIR.zip -d /opt/
RUN  unzip /opt/recon_test_pack.zip -d /opt/

# General compilation related settings.
ENV CFLAGS -O2 -D_FORTIFY_SOURCE=2 -fstack-protector --param ssp-buffer-size=4
ENV CXXFLAGS -O2 -D_FORTIFY_SOURCE=2 -fstack-protector --param ssp-buffer-size=4

# Building ECAT
RUN sed -e "s|makedepend -Dunix|makedepend -Dlinux -I /usr/include/ -I /usr/include/linux/ -I /usr/include/x86_64-linux-gnu/ -I /usr/lib/gcc/x86_64-linux-gnu/4.8/include/|" /opt/ecat/Makefile.unix| sed -e "s|CFLAGS = |CFLAGS = -fpic -fPIC -D_FORTIFY_SOURCE=2 -fstack-protector --param ssp-buffer-size=4 |" >/opt/ecat/Makefile
RUN make depend -C /opt/ecat
RUN make        -C /opt/ecat

# ECAT utils could be compiled, but it is not worth the effort
#RUN sed -e "s|-lsocket||" /opt/ecat/utils/Makefile.unix >/opt/ecat/utils/Makefile
#RUN make        -C /opt/ecat/utils

#
# Configuring and building STIR, enabling options in command line e.g. ECAT, OpenMP,
#
RUN cd /opt/STIR-bin/Release && cmake -D LLN_LIBRARIES=/opt/ecat/libecat.a -D DISABLE_LLN_MATRIX=OFF -D LLN_INCLUDE_DIRS=/opt/ecat -D STIR_OPENMP=ON -D STIR_MPI=OFF -D DISABLE_AVW=ON  -D DISABLE_RDF=ON -D DISABLE_STIR_LOCAL=ON -D BUILD_SHARED_LIBS=OFF -D BUILD_SWIG_PYTHON=OFF  /opt/STIR

# You can use different amount of cores, 
RUN make -j2  VERBOSE=1 -C /opt/STIR-bin/Release
RUN make -j2  VERBOSE=1 -C /opt/STIR-bin/Release  install
RUN make -j2  VERBOSE=1 -C /opt/STIR-bin/Release  test

#
# Test the actual compilation with the benchmarks:
#

#RUN cd /opt/STIR/recon_test_pack && ./run_ecat_tests.sh
#RUN cd /opt/STIR/recon_test_pack && ./run_test_simulate_and_recon.sh  
#RUN cd /opt/STIR/recon_test_pack && ./run_test_simulate_and_recon_with_motion.sh  
#RUN cd /opt/STIR/recon_test_pack && ./run_scatter_tests.sh  
#RUN cd /opt/STIR/recon_test_pack && ./run_tests.sh
