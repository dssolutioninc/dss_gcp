# Matlab 9.7 R2019b 稼働RuntimeのDockerイメージビルド

*Matlab 9.7 R2019b は 2019年11月リリースで、この記事投稿時点は最新バージョンとなっています。*


### Dockerfile準備

```sh:Dockerfile.matlab.R2019b
# Download and install Matlab Compiler Runtime v9.7 (2019b)
#
# This docker file will configure an environment into which the Matlab compiler
# runtime will be installed and in which stand-alone matlab routines (such as
# those created with Matlab's deploytool) can be executed.
#
# See http://www.mathworks.com/products/compiler/mcr/ for more info.

FROM debian:10-slim

ENV DEBIAN_FRONTEND noninteractive
RUN apt-get -q update && \
    apt-get install -q -y --no-install-recommends \
    xorg \
      unzip \
      wget && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
    
# Install the MCR dependencies and some things we'll need and download the MCR
# from Mathworks -silently install it
RUN mkdir /mcr-install && \
    mkdir /opt/mcr && \
    cd /mcr-install && \
    wget -q http://ssd.mathworks.com/supportfiles/downloads/R2019b/Release/2/deployment_files/installer/complete/glnxa64/MATLAB_Runtime_R2019b_Update_2_glnxa64.zip && \
    unzip -q MATLAB_Runtime_R2019b_Update_2_glnxa64.zip && \
    rm -f MATLAB_Runtime_R2019b_Update_2_glnxa64.zip && \
    ./install -destinationFolder /opt/mcr -agreeToLicense yes -mode silent && \
    cd / && \
    rm -rf mcr-install
```

Matlab runtimeパス　/opt/mcr/v97

### Dockerfileイメージビルド

```sh
# タグ指定でイメージビルド
docker build -t matlab-r2019b:latest -f Dockerfile.matlab.R2019b .
```

<br>
ご覧して頂き、どうも有難う御座います!
DSS Ben