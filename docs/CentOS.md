# Cent OS 8

### Command

```bash
$ sudo yum update

$ sudo yum install -y python3

$ sudo dnf install -y python3.12
```

### Nginx

- <https://nginx.org/en/linux_packages.html#RHEL>

```bash
$ sudo yum install -y yum-utils gd gd-devel perl libxml2 libxslt-devel gcc

# 安装 libmaxmainddb
$ wget https://github.com/maxmind/libmaxminddb/releases/download/1.9.1/libmaxminddb-1.9.1.tar.gz
$ tar -C /opt/ -xzvf libmaxminddb-1.9.1.tar.gz
$ cd /opt/libmaxminddb-1.9.1
$ ./configure
$ make
$ sudo make install
$ sudo ldconfig

# 下载 GeoIP 数据库
$ cd /opt
$ wget -O GeoLite2-City.tar.gz 'https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-City&license_key=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&suffix=tar.gz'
$ tar -zxf GeoLite2-City.tar.gz -C /opt/GeoIP/
$ wget -O GeoLite2-Country.tar.gz 'https://download.maxmind.com/app/geoip_download?edition_id=GeoLite2-Country&license_key=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&suffix=tar.gz'
$ tar -zxf GeoLite2-Country.tar.gz -C /opt/GeoIP/
$ sudo mv /opt/GeoIP/*/*.mmdb /opt/GeoIP && rm -rf /opt/GeoIP/*/

# 源码编译
# https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#sources

# 安装 pcre
$ cd /opt
wget github.com/PCRE2Project/pcre2/releases/download/pcre2-10.42/pcre2-10.42.tar.gz
tar -zxf pcre2-10.42.tar.gz
cd pcre2-10.42
./configure
make
sudo make install

## 安装 zlib
#$ cd /opt
#wget http://zlib.net/zlib-1.2.13.tar.gz
#tar -zxf zlib-1.2.13.tar.gz
#cd zlib-1.2.13
#./configure
#make
#sudo make install

# 下载 openssl
$ cd /opt
wget http://www.openssl.org/source/openssl-1.1.1v.tar.gz
tar -zxf openssl-1.1.1v.tar.gz
cd openssl-1.1.1v
./Configure darwin64-x86_64-cc --prefix=/usr
make
sudo make install

# 下载 nginx geoip 模块
# TODO 如何解压至指定目录？
$ cd /opt
wget https://github.com/leev/ngx_http_geoip2_module/archive/refs/tags/3.4.tar.gz -O ngx_http_geoip2_module.tar.gz
tar -zxf ngx_http_geoip2_module.tar.gz

# 安装 nginx
$ cd /opt
wget https://nginx.org/download/nginx-1.24.0.tar.gz
tar -zxf nginx-1.24.0.tar.gz
cd nginx-1.24.0

./configure \
  --with-cc-opt='-g -O2 -fPIE -fstack-protector-strong -Wformat -Werror=format-security -fPIC -Wdate-time -D_FORTIFY_SOURCE=2' \
  --with-ld-opt='-Wl,-Bsymbolic-functions -fPIE -pie -Wl,-z,relro -Wl,-z,now -fPIC' \
  --sbin-path=/usr/sbin/nginx \
  --conf-path=/etc/nginx/nginx.conf \
  --http-log-path=/var/log/nginx/access.log \
  --error-log-path=/var/log/nginx/error.log \
  --lock-path=/var/lock/nginx.lock \
  --pid-path=/run/nginx.pid \
  --modules-path=/usr/lib/nginx/modules \
  --http-client-body-temp-path=/var/lib/nginx/body \
  --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
  --http-proxy-temp-path=/var/lib/nginx/proxy \
  --http-scgi-temp-path=/var/lib/nginx/scgi \
  --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
  --with-compat --with-debug --with-pcre-jit \
  --with-http_ssl_module --with-http_stub_status_module \
  --with-http_realip_module --with-http_auth_request_module \
  --with-http_v2_module --with-http_dav_module \
  --with-http_slice_module --with-threads --with-http_addition_module \
  --with-http_gunzip_module \
  --with-http_gzip_static_module --with-http_image_filter_module=dynamic \
  --with-http_sub_module --with-http_xslt_module=dynamic \
  --with-stream=dynamic --with-stream_ssl_module \
  --with-stream_ssl_preread_module --with-mail=dynamic \
  --with-mail_ssl_module \
  --with-openssl=/opt/openssl-1.1.1v \
  --add-dynamic-module=/opt/ngx_http_geoip2_module

make
sudo make install
```
