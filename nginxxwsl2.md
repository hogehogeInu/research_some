## 開発工程

### WSL の導入

1. wsl->wsl2 のアップデート
1. wsl2 に CentOS を導入する
1. CentOS を MiracleLinux に置き換える
1. パッケージをアップデートする
1. wsl2 内で systemd のデーモン化（Windows サービス化）をする
1. wsl2 の USB ドライバを作成する

### wsl2 へアップデート

- [WSL2 のインストール方法](https://ari23.hatenablog.com/entry/wsl2-install)

### wsl2 に CentOS を導入

- [wsldl](https://github.com/yuk7/wsldl):Advanced WSL Distribution Launcher / Installer
- [Project List Using wsldl](https://wsldl-pg.github.io/docs/Using-wsldl/#distros)
  - Windows 10 1709 Fall Creators Update or later(x64/arm64).
  - Windows Subsystem for Linux feature is enabled.
- [wsl2 上で無料で CentOS8 を動かそう](https://www.geekfeed.co.jp/geekblog/install_centos8_on_wsl2_for_free)

1. CentOS 用 wsldl をダウンロード
   https://github.com/yuk7/CentWSL/releases/download/8.1.1911.0/CentOS8.zip
1. インストール先を作成
   `%userprofile%\AppData\Local\Packages`
1. rootfs をインポートする
   `wsl --import CentOS8 "%userprofile%\AppData\Local\Packages\CentOS8" "C:\Users\uchi\Desktop\rootfs.tar" --version 2`
1. 上でつけた wsl 名指定で起動する
   `wsl -d CentOS8`

### CentOs を MiracleLinux に置き換える

- [CentOS Linux 8 から MIRACLE LINUX 8 への移行ツールを試す](https://miraclelinux.hatenablog.com/entry/2021/10/08/120830)

```
curl -OL https://repo.dist.miraclelinux.net/miraclelinux/migration-tool/migrate2ml.sh
./migrate2ml.sh --core
dnf update -y
```

### systemd デーモン化

- [SPEC ファイルの記述](https://vinelinux.org/docs/vine6/making-rpm/make-spec.html)

* [WSL2 上の CentOS8 で systemctl を使えるようにする](https://qiita.com/jpl_ttsneo/items/2a3623dffb8fa3aed454)
* [WSL2 で Systemd を使うハック](https://qiita.com/matarillo/items/f036a9561a4839275e5f)

```
rpmbuild -bb daemonize-release-1.7.8/daemonize-1.7.8.spec
dnf install rpmbuild/RPMS/x86_64/daemonize-1.7.8-1.x86_64.rpm
history|grep dnf
dnf install https://github.com/arkane-systems/genie/releases/download/v1.44/genie-1.44-1.fc34.x86_64.rpm
genie -s
```

> Summary: A tool to run a command as a daemon
> Name: daemonize
> Version: 1.7.8
> Release: 1%{?\_dist_release}
> License: BSD
> Group: System Environment/Base
> Source: https://github.com/bmc/daemonize/archive/refs/tags/release-1.7.8.tar.gz
> BuildRoot: /var/tmp/%{name}-buildroot
> %description
> daemonize is a command-line utility that runs a command as a Unix daemon. See the accompanying man page for full details.
>
> # パッケージの作成時に必要となる情報
>
> BuildRoot: %{\_tmppath}/%{name}-%{version}-root
> Source: %{name}-%{version}.tar.gz
>
> %prep #rpm を構築する前の準備です。
> #cp -rf /root/rpmbuild/BUILD/\* $RPM_BUILD_ROOT
>
> #cd /root/rpmbuild/BUILD/
> #rm -rf /root/rpmbuild/BUILD/_
> #wget https://github.com/bmc/daemonize/archive/refs/tags/release-1.7.8.tar.gz
> #tar zfx release-1.7.8.tar.gz
> #mv daemonize-release-1.7.8/_ ./
>
> %build
> #cd /root/rpmbuild/BUILD/
> #sh configure
> #make
>
> #
>
> %install
> mkdir -p /root/rpmbuild/BUILDROOT/daemonize-1.7.8-1.x86_64/usr/local/sbin/
> mkdir -p /root/rpmbuild/BUILDROOT/daemonize-1.7.8-1.x86_64/usr/local/share/man/man1/
> /bin/cp /root/rpmbuild/BUILD/daemonize /root/rpmbuild/BUILDROOT/daemonize-1.7.8-1.x86_64/usr/local/sbin/
> /bin/cp /root/rpmbuild/BUILD/daemonize.1.gz /root/rpmbuild/BUILDROOT/daemonize-1.7.8-1.x86_64/usr/local/share/man/man1/
>
> #cd /root/rpmbuild/BUILD/
> #mkdir -p /usr/local/share/man/man1
> #mkdir -p /usr/local/sbin
> #install -c -m 0755 daemonize /usr/local/sbin
> #install -c -m 0644 ./daemonize.1 /usr/local/share/man/man1
>
> %clean
> #rm -rf /root/rpmbuild/BUILD/\*
>
> %files
> %defattr(-,root,root)
> %doc README.md CHANGELOG.md LICENSE.md
> /usr/local/sbin/daemonize
> /usr/local/share/man/man1/daemonize.1.gz

### wsl2 の USB ドライバ

- http://ktkr3d.github.io/2020/07/06/USB-support-to-WSL2/

```
dnf install build-essential flex bison libssl-dev libelf-dev libncurses-dev autoconf libudev-dev libtool
dnf upgrade gcc gcc-c++ kernel-devel systemd-devel
git clone https://github.com/microsoft/WSL2-Linux-Kernel.git /usr/src/microsoft-standard
cd /usr/src/microsoft-standard/

sudo make -j 6 && sudo make modules_install -j 6 && sudo make install -j 6
cd tools/usb/usbip
sudo ./autogen.sh
sudo ./configure
sudo sed 's/-Werror//g' -i Makefile
sudo sed 's/-Werror//g' -i src/Makefile
sudo sed 's/-Werror//g' -i libsrc/Makefile
sudo make install -j 6
sudo cp libsrc/.libs/libusbip.so.0 /lib/libusbip.so.0
```

- 疑問
  Hyper-V マネジメント画面が起動できない理由は不明

1. nginx を rpm 取得する

- [Nginx を CentOS 7 にインストールする手順](https://weblabo.oscasierra.net/nginx-centos7-install/)
- [nginx: Linux packages](https://nginx.org/en/linux_packages.html)
- [Nginx Doc](https://nginx.org/en/docs/)
  ![](2021-11-16-23-46-18.png)

* アプリケーション インフラストラクチャを Web サーバーから分離する：LAMP(Linux、Apache、MySQL、PHP、Python、Perl)ベースから脱却
* 同時実行性、遅延処理、SSL (セキュア ソケット レイヤー)
* 静的コンテンツ、圧縮とキャッシュ、接続と要求の調整
* HTTP メディア ストリーミングを軽減できる
* memcached/Redis やその他の「NoSQL」ソリューション
* [aosabook](http://www.aosabook.org/en/nginx.html)

```
vim /etc/yum.repos.d/nginx.repo
# [nginx]
# name=nginx repo
# baseurl=http://nginx.org/packages/centos/8/$basearch/
# gpgcheck=0
# enabled=1
dnf install nginx

```

1. nginx を再ビルドして、

```
  dnf install -y git gcc pcre-devel openssl-devel wget
  wget https://nginx.org/download/nginx-1.14.2.tar.gz
  tar zxfv nginx-1.14.2.tar.gz
  git clone https://github.com/arut/nginx-rtmp-module.git
  cd nginx-1.14.2/
  mv ~/nginx-rtmp-module ./
  ./configure --add-module=nginx-rtmp-module/
  make
  make install
  ls /usr/local/nginx/conf/
  vim /usr/local/nginx/conf/nginx.conf
```

1. nginx を調整して rtpm を導入する

```
sudo ln -s /usr/lib64/nginx/modules /etc/nginx/modules
mkdir -p /home/nginx
sudo useradd -r -d /var/cache/nginx/ -s /sbin/nologin -U nginx
chown -R nginx:nginx /home/nginx
git clone https://github.com/Intel-Media-SDK/MediaSDK.git msdk
cd msdk/ && mkdir build && cd build && cmake .. && make
```

- [RTMP モジュールを使用](https://www.xlsoft.com/jp/blog/blog/2019/11/02/post-7858/)

## 外部 dnf

- [MIRACLE LINUX 8.4 でいろんなリポジトリ](https://www.cybertrust.co.jp/blog/linux-oss/linux/miraclelinux84-repository.html)

```
dnf config-manager --set-enabled 8-latest-PowerTools
dnf config-manager --set-enabled 8-latest-HighAvailability
dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
dnf install http://rpms.remirepo.net/enterprise/remi-release-8.rpm
```

## 気になるドキュメント

- [【JavaScript】Web カメラの映像をブラウザに表示する方法](https://reffect.co.jp/html/javascript-webcamera)
  - `audio:true`にて音声もキャプチャできる
