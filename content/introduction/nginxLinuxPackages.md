+++
title = "nginx: Linux packages"
date = 2023-08-14T17:08:39+08:00
weight = 11
type = "docs"
description = ""
isCJKLanguage = true
draft = false

+++

# nginx: Linux packages

https://nginx.org/en/linux_packages.html

## Supported distributions and versions

nginx packages are available for the following Linux distributions and versions:

[RHEL and derivatives](https://nginx.org/en/linux_packages.html#RHEL)

> | Version | Supported Platforms          |
> | ------- | ---------------------------- |
> | 7.4+    | x86_64, aarch64/arm64        |
> | 8.x     | x86_64, aarch64/arm64, s390x |
> | 9.x     | x86_64, aarch64/arm64, s390x |



[Debian](https://nginx.org/en/linux_packages.html#Debian)

> | Version         | Supported Platforms   |
> | --------------- | --------------------- |
> | 11.x “bullseye” | x86_64, aarch64/arm64 |
> | 12.x “bookworm” | x86_64, aarch64/arm64 |



[Ubuntu](https://nginx.org/en/linux_packages.html#Ubuntu)

> | Version         | Supported Platforms          |
> | --------------- | ---------------------------- |
> | 20.04 “focal”   | x86_64, aarch64/arm64, s390x |
> | 22.04 “jammy”   | x86_64, aarch64/arm64, s390x |
> | 22.10 “kinetic” | x86_64, aarch64/arm64        |
> | 23.04 “lunar”   | x86_64, aarch64/arm64        |



[SLES](https://nginx.org/en/linux_packages.html#SLES)

> | Version | Supported Platforms |
> | ------- | ------------------- |
> | 12 SP5+ | x86_64              |
> | 15 SP2+ | x86_64              |



[Alpine](https://nginx.org/en/linux_packages.html#Alpine)

> | Version | Supported platforms   |
> | ------- | --------------------- |
> | 3.15    | x86_64, aarch64/arm64 |
> | 3.16    | x86_64, aarch64/arm64 |
> | 3.17    | x86_64, aarch64/arm64 |
> | 3.18    | x86_64, aarch64/arm64 |



[Amazon Linux](https://nginx.org/en/linux_packages.html#Amazon-Linux)

> | Version | Supported platforms   |
> | ------- | --------------------- |
> | 2 (LTS) | x86_64, aarch64/arm64 |
> | 2023    | x86_64, aarch64/arm64 |



Packages for RHEL 7 and SLES 12 are built without [HTTP/3 support](https://nginx.org/en/docs/http/ngx_http_v3_module.html) because OpenSSL used by those doesn't support TLSv1.3.



## Installation instructions

Before you install nginx for the first time on a new machine, you need to set up the nginx packages repository. Afterward, you can install and update nginx from the repository.



## RHEL and derivatives

This section applies to Red Hat Enterprise Linux and its derivatives such as CentOS, Oracle Linux, Rocky Linux, AlmaLinux.

Install the prerequisites:

> ```
> sudo yum install yum-utils
> ```

To set up the yum repository, create the file named `/etc/yum.repos.d/nginx.repo` with the following contents:

> ```
> [nginx-stable]
> name=nginx stable repo
> baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
> gpgcheck=1
> enabled=1
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> 
> [nginx-mainline]
> name=nginx mainline repo
> baseurl=http://nginx.org/packages/mainline/centos/$releasever/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> ```

By default, the repository for stable nginx packages is used. If you would like to use mainline nginx packages, run the following command:

> ```
> sudo yum-config-manager --enable nginx-mainline
> ```

To install nginx, run the following command:

> ```
> sudo yum install nginx
> ```

When prompted to accept the GPG key, verify that the fingerprint matches `573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`, and if so, accept it.



## Debian

Install the prerequisites:

> ```
> sudo apt install curl gnupg2 ca-certificates lsb-release debian-archive-keyring
> ```



Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:

> ```
> curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
>     | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
> ```

Verify that the downloaded file contains the proper key:

> ```
> gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
> ```

The output should contain the full fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` as follows:

> ```
> pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
>       573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
> uid                      nginx signing key <signing-key@nginx.com>
> ```

If the fingerprint is different, remove the file.

To set up the apt repository for stable nginx packages, run the following command:

> ```
> echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
> http://nginx.org/packages/debian `lsb_release -cs` nginx" \
>     | sudo tee /etc/apt/sources.list.d/nginx.list
> ```

If you would like to use mainline nginx packages, run the following command instead:

> ```
> echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
> http://nginx.org/packages/mainline/debian `lsb_release -cs` nginx" \
>     | sudo tee /etc/apt/sources.list.d/nginx.list
> ```



Set up repository pinning to prefer our packages over distribution-provided ones:

> ```
> echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
>     | sudo tee /etc/apt/preferences.d/99nginx
> ```



To install nginx, run the following commands:

> ```
> sudo apt update
> sudo apt install nginx
> ```





## Ubuntu

Install the prerequisites:

> ```
> sudo apt install curl gnupg2 ca-certificates lsb-release ubuntu-keyring
> ```



Import an official nginx signing key so apt could verify the packages authenticity. Fetch the key:

> ```
> curl https://nginx.org/keys/nginx_signing.key | gpg --dearmor \
>     | sudo tee /usr/share/keyrings/nginx-archive-keyring.gpg >/dev/null
> ```

Verify that the downloaded file contains the proper key:

> ```
> gpg --dry-run --quiet --no-keyring --import --import-options import-show /usr/share/keyrings/nginx-archive-keyring.gpg
> ```

The output should contain the full fingerprint `573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62` as follows:

> ```
> pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
>       573BFD6B3D8FBC641079A6ABABF5BD827BD9BF62
> uid                      nginx signing key <signing-key@nginx.com>
> ```

If the fingerprint is different, remove the file.

To set up the apt repository for stable nginx packages, run the following command:

> ```
> echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
> http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \
>     | sudo tee /etc/apt/sources.list.d/nginx.list
> ```

If you would like to use mainline nginx packages, run the following command instead:

> ```
> echo "deb [signed-by=/usr/share/keyrings/nginx-archive-keyring.gpg] \
> http://nginx.org/packages/mainline/ubuntu `lsb_release -cs` nginx" \
>     | sudo tee /etc/apt/sources.list.d/nginx.list
> ```



Set up repository pinning to prefer our packages over distribution-provided ones:

> ```
> echo -e "Package: *\nPin: origin nginx.org\nPin: release o=nginx\nPin-Priority: 900\n" \
>     | sudo tee /etc/apt/preferences.d/99nginx
> ```



To install nginx, run the following commands:

> ```
> sudo apt update
> sudo apt install nginx
> ```





## SLES

Install the prerequisites:

> ```
> sudo zypper install curl ca-certificates gpg2
> ```

To set up the zypper repository for stable nginx packages, run the following command:

> ```
> sudo zypper addrepo --gpgcheck --type yum --refresh --check \
>     'http://nginx.org/packages/sles/$releasever_major' nginx-stable
> ```

If you would like to use mainline nginx packages, run the following command instead:

> ```
> sudo zypper addrepo --gpgcheck --type yum --refresh --check \
>     'http://nginx.org/packages/mainline/sles/$releasever_major' nginx-mainline
> ```

Next, import an official nginx signing key so zypper/rpm could verify the packages authenticity. Fetch the key:

> ```
> curl -o /tmp/nginx_signing.key https://nginx.org/keys/nginx_signing.key
> ```

Verify that the downloaded file contains the proper key:

> ```
> gpg --with-fingerprint /tmp/nginx_signing.key
> ```

The output should contain the full fingerprint `573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62` as follows:

> ```
> pub  2048R/7BD9BF62 2011-08-19 [expires: 2024-06-14]
>       Key fingerprint = 573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
> uid nginx signing key <signing-key@nginx.com>
> ```

Finally, import the key to the rpm database:

> ```
> sudo rpmkeys --import /tmp/nginx_signing.key
> ```

To install nginx, run the following command:

> ```
> sudo zypper install nginx
> ```





## Alpine

Install the prerequisites:

> ```
> sudo apk add openssl curl ca-certificates
> ```

To set up the apk repository for stable nginx packages, run the following command:

> ```
> printf "%s%s%s%s\n" \
>     "@nginx " \
>     "http://nginx.org/packages/alpine/v" \
>     `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
>     "/main" \
>     | sudo tee -a /etc/apk/repositories
> ```

If you would like to use mainline nginx packages, run the following command instead:

> ```
> printf "%s%s%s%s\n" \
>     "@nginx " \
>     "http://nginx.org/packages/mainline/alpine/v" \
>     `egrep -o '^[0-9]+\.[0-9]+' /etc/alpine-release` \
>     "/main" \
>     | sudo tee -a /etc/apk/repositories
> ```

Next, import an official nginx signing key so apk could verify the packages authenticity. Fetch the key:

> ```
> curl -o /tmp/nginx_signing.rsa.pub https://nginx.org/keys/nginx_signing.rsa.pub
> ```

Verify that downloaded file contains the proper key:

> ```
> openssl rsa -pubin -in /tmp/nginx_signing.rsa.pub -text -noout
> ```

The output should contain the following modulus:

> ```
> Public-Key: (2048 bit)
> Modulus:
>     00:fe:14:f6:0a:1a:b8:86:19:fe:cd:ab:02:9f:58:
>     2f:37:70:15:74:d6:06:9b:81:55:90:99:96:cc:70:
>     5c:de:5b:e8:4c:b2:0c:47:5b:a8:a2:98:3d:11:b1:
>     f6:7d:a0:46:df:24:23:c6:d0:24:52:67:ba:69:ab:
>     9a:4a:6a:66:2c:db:e1:09:f1:0d:b2:b0:e1:47:1f:
>     0a:46:ac:0d:82:f3:3c:8d:02:ce:08:43:19:d9:64:
>     86:c4:4e:07:12:c0:5b:43:ba:7d:17:8a:a3:f0:3d:
>     98:32:b9:75:66:f4:f0:1b:2d:94:5b:7c:1c:e6:f3:
>     04:7f:dd:25:b2:82:a6:41:04:b7:50:93:94:c4:7c:
>     34:7e:12:7c:bf:33:54:55:47:8c:42:94:40:8e:34:
>     5f:54:04:1d:9e:8c:57:48:d4:b0:f8:e4:03:db:3f:
>     68:6c:37:fa:62:14:1c:94:d6:de:f2:2b:68:29:17:
>     24:6d:f7:b5:b3:18:79:fd:31:5e:7f:4c:be:c0:99:
>     13:cc:e2:97:2b:dc:96:9c:9a:d0:a7:c5:77:82:67:
>     c9:cb:a9:e7:68:4a:e1:c5:ba:1c:32:0e:79:40:6e:
>     ef:08:d7:a3:b9:5d:1a:df:ce:1a:c7:44:91:4c:d4:
>     99:c8:88:69:b3:66:2e:b3:06:f1:f4:22:d7:f2:5f:
>     ab:6d
> Exponent: 65537 (0x10001)
> ```

Finally, move the key to apk trusted keys storage:

> ```
> sudo mv /tmp/nginx_signing.rsa.pub /etc/apk/keys/
> ```

To install nginx, run the following command:

> ```
> sudo apk add nginx@nginx
> ```

The `@nginx` tag should also be specified when installing packages with [dynamic modules](https://nginx.org/en/linux_packages.html#dynmodules):

> ```
> sudo apk add nginx-module-image-filter@nginx nginx-module-njs@nginx
> ```





## Amazon Linux

Install the prerequisites:

> ```
> sudo yum install yum-utils
> ```

To set up the yum repository for Amazon Linux 2, create the file named `/etc/yum.repos.d/nginx.repo` with the following contents:

> ```
> [nginx-stable]
> name=nginx stable repo
> baseurl=http://nginx.org/packages/amzn2/$releasever/$basearch/
> gpgcheck=1
> enabled=1
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> priority=9
> 
> [nginx-mainline]
> name=nginx mainline repo
> baseurl=http://nginx.org/packages/mainline/amzn2/$releasever/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> priority=9
> ```

To set up the yum repository for Amazon Linux 2023, create the file named `/etc/yum.repos.d/nginx.repo` with the following contents:

> ```
> [nginx-stable]
> name=nginx stable repo
> baseurl=http://nginx.org/packages/amzn/2023/$basearch/
> gpgcheck=1
> enabled=1
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> priority=9
> 
> [nginx-mainline]
> name=nginx mainline repo
> baseurl=http://nginx.org/packages/mainline/amzn/2023/$basearch/
> gpgcheck=1
> enabled=0
> gpgkey=https://nginx.org/keys/nginx_signing.key
> module_hotfixes=true
> priority=9
> ```

By default, the repository for stable nginx packages is used. If you would like to use mainline nginx packages, run the following command:

> ```
> sudo yum-config-manager --enable nginx-mainline
> ```

To install nginx, run the following command:

> ```
> sudo yum install nginx
> ```

When prompted to accept the GPG key, verify that the fingerprint matches `573B FD6B 3D8F BC64 1079 A6AB ABF5 BD82 7BD9 BF62`, and if so, accept it.



## Source Packages

Packaging sources can be found in the [packaging sources repository](http://hg.nginx.org/pkg-oss).

The `default` branch holds packaging sources for the current mainline version, while `stable-*` branches contain latest sources for stable releases. To build binary packages, run `make` in `debian/` directory on Debian/Ubuntu, or in `rpm/SPECS/` on RHEL and derivatives, SLES, and Amazon Linux, or in `alpine/` on Alpine.

Packaging sources are distributed under the same [2-clause BSD-like license](https://nginx.org/LICENSE) used by nginx.



## Dynamic Modules

Main nginx package is built with all modules that do not require additional libraries to avoid extra dependencies. Since version 1.9.11, nginx supports [dynamic modules](https://nginx.org/en/docs/ngx_core_module.html#load_module) and the following modules are built as dynamic and shipped as separate packages:

> ```
> nginx-module-geoip
> nginx-module-image-filter
> nginx-module-njs
> nginx-module-perl
> nginx-module-xslt
> ```





## Signatures

Since our [PGP keys](https://nginx.org/en/pgp_keys.html) and packages are located on the same server, they are equally trusted. It is highly advised to additionally verify the authenticity of the downloaded PGP key. PGP has the “Web of Trust” concept, when a key is signed by someone else’s key, that in turn is signed by another key and so on. It often makes possible to build a chain from an arbitrary key to someone’s key who you know and trust personally, thus verify the authenticity of the first key in a chain. This concept is described in details in [GPG Mini Howto](https://www.gnupg.org/howtos/en/GPGMiniHowto-1.html). Our keys have enough signatures, and their authenticity is relatively easy to check.