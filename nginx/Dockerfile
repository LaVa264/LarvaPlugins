FROM ubuntu:22.04

#Docker Hangs after selecting 'Geographic area': https://github.com/Netflix/security_monkey/issues/1197
ARG DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get -y upgrade && \
    apt-get install -y libssl-dev apt-utils autoconf automake && \
    apt-get install -y build-essential libcurl4-openssl-dev libgeoip-dev liblmdb-dev libpcre++-dev libtool &&\
    apt-get install -y libxml2-dev libyajl-dev pkgconf zlib1g-dev libpcre2-dev && \
    apt-get install -y bison flex gcc g++ make && \
    apt-get install -y dnsutils vim coreutils gpg

RUN apt-get install -y wget

ADD ./modules/ /tmp/modules/

# Install dynamic modules.
WORKDIR /tmp/modules/

RUN wget https://github.com/SpiderLabs/ModSecurity-nginx/releases/download/v1.0.3/modsecurity-nginx-v1.0.3.tar.gz
RUN tar -xzvf modsecurity-nginx-v1.0.3.tar.gz

# Install modsecurity library.
WORKDIR /tmp/

RUN wget https://github.com/SpiderLabs/ModSecurity/releases/download/v3.0.8/modsecurity-v3.0.8.tar.gz
RUN tar -xvf modsecurity-v3.0.8.tar.gz

WORKDIR /tmp/modsecurity-v3.0.8/
RUN ./build.sh && ./configure && make -j4 && make install

# Install core rule set.
WORKDIR /tmp/
RUN wget https://github.com/coreruleset/coreruleset/archive/refs/tags/v3.3.4.tar.gz
RUN tar -xzvf v3.3.4.tar.gz

# Build nginx from source.
WORKDIR /tmp/
RUN wget http://nginx.org/download/nginx-1.23.3.tar.gz
RUN tar -xzvf nginx-1.23.3.tar.gz

WORKDIR /tmp/nginx-1.23.3/

RUN ./configure \
    --sbin-path=/usr/nginx/sbin/nginx \
    --conf-path=/usr/nginx/conf/nginx.conf \
    --error-log-path=/usr/nginx/logs/error.log \
    --http-log-path=/usr/nginx/logs/access.log \
    --pid-path=/usr/nginx/logs/nginx.pid \
    --prefix=/usr/nginx/conf \
    --modules-path=/usr/nginx/modules \
    --with-pcre \
    --with-http_ssl_module \
    --with-http_v2_module \
    --with-http_geoip_module \
    --with-http_stub_status_module \
    --user=root \
    --group=root \
    --add-dynamic-module=/tmp/modules/modsecurity-nginx-v1.0.3 \
    --without-http_scgi_module \
    --without-http_grpc_module \
    --without-http_empty_gif_module \
    --without-http_upstream_random_module \
    --without-mail_pop3_module \
    --without-mail_imap_module \
    --without-mail_smtp_module \
    --without-stream_limit_conn_module \
    --without-stream_access_module \
    --without-stream_geo_module \
    --without-stream_map_module \
    --without-stream_split_clients_module \
    --without-stream_return_module \
    --without-stream_set_module \
    --without-stream_upstream_hash_module \
    --without-stream_upstream_least_conn_module \
    --without-stream_upstream_random_module \
    --without-stream_upstream_zone_module

RUN make -j4 && make install

ADD ./conf/nginx.conf /usr/nginx/conf/
ADD ./conf/modules/ /usr/nginx/conf/

ADD ./conf/rules/ /usr/nginx/conf/

WORKDIR /usr/nginx/conf/rules


RUN mkdir -p /data/nginx/cache/

WORKDIR /home/

CMD ["/usr/nginx/sbin/nginx", "-g" ,"daemon off;"]
