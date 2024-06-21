# Use a base image with the necessary tools for building software
FROM ubuntu:20.04

# Set environment variables
ENV OPENRESTY_VERSION=1.21.4.1
ENV DEBIAN_FRONTEND=noninteractive

# Install required dependencies
RUN apt-get update && \
    apt-get install -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    perl \
    make \
    curl \
    zlib1g-dev \
    libiconv-hook-dev \
    libpq-dev \
    libpcre3 \
    libssl1.1 \
    zlib1g \
    libreadline-dev \
    libncurses5-dev \
    libncursesw5-dev \
    libpcre++-dev \
    libtool \
    autoconf \
    git

# Download and extract OpenResty
RUN curl -fSL https://openresty.org/download/openresty-${OPENRESTY_VERSION}.tar.gz -o openresty.tar.gz && \
    tar -xzf openresty.tar.gz && \
    rm openresty.tar.gz

# Build and install OpenResty with the specified configure options
RUN cd openresty-${OPENRESTY_VERSION} && \
    ./configure --prefix=/opt/openresty \
                --with-pcre-jit \
                --with-ipv6 \
                --without-http_redis2_module \
                --with-http_iconv_module \
                --with-http_postgres_module \
                -j8 && \
    make -j8 && \
    make install

# Clean up the build dependencies to reduce image size
RUN apt-get remove -y \
    build-essential \
    libpcre3-dev \
    libssl-dev \
    perl \
    make \
    zlib1g-dev \
    libiconv-hook-dev \
    libpq-dev && \
    apt-get autoremove -y && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Set the PATH environment variable to include OpenResty binaries
ENV PATH=/opt/openresty/bin:/opt/openresty/nginx/sbin:$PATH

# Expose the default HTTP and HTTPS ports
EXPOSE 80 443

# Set the default command to start Nginx with OpenResty
CMD ["nginx", "-g", "daemon off;"]
