## NGINX-QAT-HARDWARE
[NGINX](https://nginx.org/) is a free, open-source, high-performance web server that can also be used as a reverse proxy, load balancer, mail proxy and HTTP cache. NGINX is known for its high performance, stability, rich feature set, simple configuration, and low resource consumption.

Nginx uses SSL/TLS to enhance web access security. Intel has introduced the Crypto-NI software solution which is based on
Intel® Xeon® Scalable Processors (Codename Ice Lake/Whitley). It can effectively improve the security of web access. [Intel_Asynch_Nginx](https://github.com/intel/asynch_mode_nginx.git) is an Intel optimized version Nginx,  used by Intel to support Async hardware and software acceleration for https.

The main software used in this solution are IPP Cryptography Library, Intel Multi-Buffer Crypto for IPsec Library ([intel-ipsec-mb](https://github.com/intel/intel-ipsec-mb.git)) and Intel®  QuickAssist Technology ([Intel® QAT](https://github.com/intel/QAT_Engine.git)), which provide batch submission of multiple SSL requests and parallel asynchronous processing mechanism based on the new instruction set, greatly improving the performance. Intel® QuickAssist Accelerator is a PCIe card that needs to be inserted into the PCIe slot in the server at the start.

#nginx, #web server, #reverse proxy, #load balancer, #mail proxy, #HTTP cache

## Software Components
Table 1 lists the necessary software components. 
The descending row order represents the install sequence. 
The recommended component version and download location are also provided.

Table 1: Software Components

| Component| Version |
| :---        |    :----:   |
| UBUNTU | [v22.04](https://ubuntu.com/) |
| git          | [1:2.34.1-1ubuntu1.8](https://github.com/git-guides/install-git#install-git-on-linux) |
| gcc          | [4:11.2.0-1ubuntu1](https://gcc.gnu.org/) |
| g++          | [4:11.2.0-1ubuntu1](https://gcc.gnu.org/) |
| cmake        | [3.22.1-1ubuntu1.22.04.1](https://cmake.org/) |
| make         | [4.3-4.1build1](https://www.gnu.org/software/make/) |
| automake     | [1:1.16.5-1.3](https://www.gnu.org/software/automake/) |
| autoconf     | [2.71-2](https://www.gnu.org/software/autoconf/) |
| libtool      | [2.4.6-15build2](https://www.gnu.org/software/libtool/) |
| nasm         | [2.15.05-1](https://www.nasm.us/) |
| yasm         | [1.3.0-2.1](https://yasm.tortall.net/) |
| perl         | [5.34.0-3ubuntu1.1](https://www.perl.org/) |
| zlib1g-dev   | [1:1.2.11.dfsg-2ubuntu9.2](https://packages.ubuntu.com/jammy/zlib1g-dev) |
| pkg-config   | [0.29.2-1ubuntu3](https://www.freedesktop.org/wiki/Software/pkg-config/) |
| OpenSSL      | [1_1_1t](https://github.com/openssl/openssl.git) |
| QAT Lib      | [23.02.0](https://github.com/intel/qatlib.git) |
| IPP crypto   | [ippcp_2021.7](https://github.com/intel/ipp-crypto) |
| Intel IPSec  | [v1.3](https://github.com/intel/intel-ipsec-mb) |
| QAT Engine   | [v0.6.19](https://github.com/intel/QAT_Engine) |
| numactl      | [2.0.14-3ubuntu2](https://github.com/numactl/numactl) |
| libprocps8   | [2:3.3.17-6ubuntu2](https://packages.ubuntu.com/jammy/libprocps8) |
| zlib1g       | [1:1.2.11.dfsg-2ubuntu9.2](https://packages.ubuntu.com/jammy/zlib1g) |
| qat-crypto-base-ssl1-ubuntu | [latest](https://hub.docker.com/u/intel) |
| ASYNC NGINX | [v0.4.7](https://github.com/intel/asynch_mode_nginx.git) |

## Configuration Snippets
This section contains code snippets on build instructions for software components.

Note: Common Linux utilities, such as docker, git, wget, will not be listed here. Please install on demand if it is not provided in base OS installation.

### UBUNTU
```sh
docker pull ubuntu:22.04
```

### qat-crypto-base-ssl1-ubuntu
```sh
docker pull qat-crypto-base-ssl1-ubuntu:latest
```
### OpenSSL
```
OPENSSL_INCLUDE_DIR="/usr/local/include/openssl"
OPENSSL_LIBRARIES_DIR="/usr/local/lib"
OPENSSL_ROOT_DIR="/usr/local/bin/openssl"
OPENSSL_VER=1_1_1t
OPENSSL_REPO=https://github.com/openssl/openssl.git

git clone --depth 1 -b OpenSSL_${OPENSSL_VER} ${OPENSSL_REPO} openssl && \
cd openssl && \
./config && \
make depend && \
make -j && \
make install
```

### QAT Lib
```
QATLIB_VER="23.02.0"
QATLIB_REPO="https://github.com/intel/qatlib.git"
git clone --depth 1 -b ${QATLIB_VER} ${QATLIB_REPO} && \
cd qatlib && \
./autogen.sh && \
./configure --prefix=/usr/local/ \ 
    --enable-systemd=no \
    --enable-service && \
make -j && \
make install && \
make samples-install 
```

### IPP Crypto
```
IPP_CRYPTO_VER="ippcp_2021.7"
IPP_CRYPTO_REPO="https://github.com/intel/ipp-crypto"
git clone --depth 1 -b ${IPP_CRYPTO_VER} ${IPP_CRYPTO_REPO} && \
cd ipp-crypto/sources/ippcp/crypto_mb && \
cmake . -B"../build" \
    -DOPENSSL_INCLUDE_DIR=${OPENSSL_INCLUDE_DIR} \
    -DOPENSSL_LIBRARIES=${OPENSSL_LIBRARIES_DIR} \
    -DOPENSSL_ROOT_DIR=${OPENSSL_ROOT_DIR} && \
cd ../build && \
make crypto_mb && make install
```

### Intel IPSec
```
IPSEC_MB_VER="v1.3"
IPSEC_MB_REPO="https://github.com/intel/intel-ipsec-mb"
git clone --depth 1 -b ${IPSEC_MB_VER} ${IPSEC_MB_REPO} && \
cd intel-ipsec-mb && \
make && make install LIB_INSTALL_DIR=/usr/local/lib
```

### QAT Engine
```
QATENGINE_VER="v0.6.19"
QATENGINE_REPO="https://github.com/intel/QAT_Engine"
git clone --depth 1 -b ${QATENGINE_VER} ${QATENGINE_REPO} && \
cd QAT_Engine && \
./autogen.sh && \
./configure \
    --enable-multibuff_offload \
    --enable-ipsec_offload \
    --enable-multibuff_ecx \
    --enable-qat_sw  && \
make && make install
```


### ASYNC NGINX
```sh
ARG ASYNC_NGINX_VER="v0.4.7"
ARG ASYNC_NGINX_REPO=https://github.com/intel/asynch_mode_nginx.git
RUN git clone -b $ASYNC_NGINX_VER --depth 1 ${ASYNC_NGINX_REPO} && \
    cd /asynch_mode_nginx && \
    ./configure \
      --prefix=/var/www \
      --conf-path=/usr/local/share/nginx/conf/nginx.conf \
      --sbin-path=/usr/local/bin/nginx \
      --pid-path=/run/nginx.pid \
      --lock-path=/run/lock/nginx.lock \
      --modules-path=/var/www/modules/ \
      --without-http_rewrite_module \
      --with-http_ssl_module \
      --with-pcre \
      --add-dynamic-module=modules/nginx_qat_module/ \
      --with-cc-opt="-DNGX_SECURE_MEM -O3 -I/usr/local/include/openssl -Wno-error=deprecated-declarations -Wimplicit-fallthrough=0" \
      --with-ld-opt="-ltcmalloc_minimal -Wl,-rpath=/usr/local/lib -L/usr/local/lib" && \
    make -j && \
    make install
```

Workload Services Framework

-end of document-