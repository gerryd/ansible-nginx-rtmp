# ansible-nginx-rtmp

This is a work in progress.

## Building nginx with the nginx-rtmp module

The resulting deb packages have been included with this role. The instructions
below are only useful should you wish to build your own.


### Dependencies

Make sure you have the source repositories configured. Your `sources.list` file
should look somewhat like this:

    deb http://deb.debian.org/debian/ jessie main
    deb-src http://deb.debian.org/debian/ jessie main
    
    deb http://deb.debian.org/debian/ jessie-updates main
    deb-src http://deb.debian.org/debian/ jessie-updates main
    
    deb http://deb.debian.org/debian-security jessie/updates main
    deb-src http://deb.debian.org/debian-security jessie/updates main

Install the build dependencies and some packages we'll need.

    apt-get update
    apt-get build-dep nginx
    apt-get install git fakeroot vim devscripts


### Get the sources

Fetch the nginx package source into your local build directory.

    mkdir -p /usr/src/nginx-rtmp/
    cd /usr/src/nginx-rtmp/
    apt-get source nginx

Go to the modules directory, and fetch the `nginx-rtmp` module and go back to
the root of the package.

    cd nginx-*/debian/modules/
    git clone https://github.com/arut/nginx-rtmp-module.git
    cd ../../


### Add the module to the build

Edit the `debian/rules` file and add the relevant `--add-module` option to the
`extras_configure_flags` section. It should look somewhat like this.

    extras_configure_flags := \
                            $(common_configure_flags) \
                            --with-http_addition_module \
                            --with-http_dav_module \
                            --with-http_flv_module \
                            --with-http_geoip_module \
                            --with-http_gzip_static_module \
                            --with-http_image_filter_module \
                            --with-http_mp4_module \
                            --with-http_perl_module \
                            --with-http_random_index_module \
                            --with-http_secure_link_module \
                            --with-http_spdy_module \
                            --with-http_sub_module \
                            --with-http_xslt_module \
                            --with-mail \
                            --with-mail_ssl_module \
                            --add-module=$(MODULESDIR)/headers-more-nginx-module \
                            --add-module=$(MODULESDIR)/nginx-auth-pam \
                            --add-module=$(MODULESDIR)/nginx-cache-purge \
                            --add-module=$(MODULESDIR)/nginx-dav-ext-module \
                            --add-module=$(MODULESDIR)/nginx-development-kit \
                            --add-module=$(MODULESDIR)/nginx-echo \
                            --add-module=$(MODULESDIR)/ngx-fancyindex \
                            --add-module=$(MODULESDIR)/nginx-http-push \
                            --add-module=$(MODULESDIR)/nginx-lua \
                            --add-module=$(MODULESDIR)/nginx-upload-progress \
                            --add-module=$(MODULESDIR)/nginx-upstream-fair \
                            --add-module=$(MODULESDIR)/ngx_http_substitutions_filter_module \
                            --add-module=$(MODULESDIR)/nginx-rtmp-module


### Build and install the package

Before building the package, you will need to update the changelog and make a
custom version of the package.

    DEBFULLNAME=Me DEBEMAIL=builder@example.com dch --nmu

Make the entry look somewhat like the one below, pay attention to the version
number.

    nginx (1.6.2-5+deb8u4-rtmp1) UNRELEASED; urgency=medium
    
      * Non-maintainer upload.
      * Add nginx-rtmp module
    
     -- Me <builder@example.com>  Sun, 12 Mar 2017 10:00:00 +0000

Build the package.

    dpkg-buildpackage -b

If you now install the resulting `nginx-common` and `nginx-extras` packages, you
should be all set.

    cd ../
    dpkg -i nginx-common_*.deb
    dpkg -i nginx-extras_*.deb
