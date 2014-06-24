# 
#  Dockerfile for GATE 7.0  
#  http://www.opengatecollaboration.org/
#
#  The image is based on the vGate 3.0 image, but it has been modified a bit, see below
#

FROM dvolgyes/vgate3_vanilia
MAINTAINER David Völgyes <david.volgyes@ieee.org>

# Setting environment variables
ENV TERM linux
ENV DEBIAN_FRONTEND noninteractive

RUN   apt-get update
RUN   apt-get install      -y --force-yes dialog debconf-utils
RUN   apt-get clean
