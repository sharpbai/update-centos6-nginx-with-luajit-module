update-centos-nginx-with-lua-support for Centos 6.x
================================

This module compile latest nginx version and complementary modules on centos 6.x
The compiled nginx is fully compatible with nginx installed from yum.
The configure parameters is the same as centos nginx installed from yum except added lua support.
You could add luajit and redis support for your centos nginx without painful config files changed.

## Installation dependence

    yum -y groupinstall "Development Tools"
    yum -y install gcc gcc-c++ cmake kernel-devel
    yum -y install zlib-devel
    yum -y install cpio
    yum -y install expat-devel
    yum -y install gettext-devel
    yum -y install libxslt
    yum -y install libxslt-devel
    yum -y install gd gd-devel
    yum -y install perl-ExtUtils-Embed
    yum -y install openssl
    yum -y install openssl-devel
    yum -y install lua-devel
    
    
## Download latest Pcre 8.34

    cd /usr/local/src/
    wget http://heanet.dl.sourceforge.net/project/pcre/pcre/8.34/pcre-8.34.tar.gz
    tar -xzvf pcre-8.34.tar.gz
    cd pcre-8.34/
    ./configure # /usr/local is the default so no need to prefix
    make
    sudo make install
    sudo ldconfig # this is important otherwise nginx will compile but fail to load
    
## Download latest stable Luajit

    cd /usr/local/src/
    git clone http://luajit.org/git/luajit-2.0.git
    
Install

    cd /usr/local/src/luajit-2.0/
    cmake .
    make install
    
## Git latest nginx

    cd /usr/local/src/
    git clone --depth 1 https://github.com/nginx/nginx

## Git ngx_devel_kit

    cd /usr/local/src/
    git clone --depth 1 https://github.com/simpl/ngx_devel_kit
    
## Git nginx-sitcky-module-ng

    cd /usr/local/src/
    git clone --depth 1 https://bitbucket.org/nginx-goodies/nginx-sticky-module-ng

## Git lua-nginx-module

    cd /usr/local/src/
    git clone --depth 1 https://github.com/openresty/lua-nginx-module
    
Update lua-nginx-module

    cd /usr/local/src/lua-nginx-module
    git pull
    
## Git lua-resty-redis

    cd /usr/local/src/
    git clone --depth 1 https://github.com/openresty/lua-resty-redis

## Export luajit path
export lua lib path

   export LUAJIT_LIB=/usr/local/lib
   export LUAJIT_INC=/usr/local/include/luajit-2.0
    
## Compile Nginx with :
lua-nginx-module
ngx_devel_kit (This module is essential. If you skip it when compiling nginx the lua module will not work)

    cd /usr/local/src/nginx
    ./auto/configure \
    --prefix=/etc/nginx \
    --sbin-path=/usr/sbin/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --error-log-path=/var/log/nginx/error.log \
    --http-log-path=/var/log/nginx/access.log \
    --pid-path=/var/run/nginx.pid \
    --lock-path=/var/run/nginx.lock \
    --http-client-body-temp-path=/var/cache/nginx/client_temp \
    --http-proxy-temp-path=/var/cache/nginx/proxy_temp \
    --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
    --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
    --http-scgi-temp-path=/var/cache/nginx/scgi_temp \
    --user=nginx \
    --group=nginx \
    --with-http_ssl_module \
    --with-http_realip_module \
    --with-http_addition_module \
    --with-http_sub_module \
    --with-http_dav_module \
    --with-http_flv_module \
    --with-http_mp4_module \
    --with-http_gunzip_module \
    --with-http_gzip_static_module \
    --with-http_random_index_module \
    --with-http_secure_link_module \
    --with-http_stub_status_module \
    --with-http_auth_request_module \
    --with-mail \
    --with-mail_ssl_module \
    --with-file-aio \
    --with-ipv6 \
    --with-http_v2_module \
    --with-cc-opt='-O2 -g -pipe -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector --param=ssp-buffer-size=4 -m64 -mtune=generic' \
    --with-ld-opt="-Wl,-rpath,/usr/local/lib" \
    --with-pcre=/usr/local/src/pcre-8.34 \
    --add-module=/usr/local/src/lua-nginx-module \
    --add-module=/usr/local/src/nginx-sticky-module-ng \
    --add-module=/usr/local/src/ngx_devel_kit
    make
    make install

## Copy lua redis lib

    cp -r /usr/local/src/lua-resty-redis/lib/resty /usr/local/share/luajit-2.0.4/
    
## Add init file

    chkconfig nginx on
    chmod a+x /etc/init.d/nginx
    service nginx start

If you have already installed nginx from yum, you should only run

    service nginx stop
    service nginx start
