*このページはメンテナンスしていないため古い情報を含んでいる

Debian/GNU Linux
================


.bashrc
-------
```bash
$ vi .bashrc
PS1="\[\033[31m\]\w/\n\[\033[31m\]\u@\h[\t]\[\033[0m\]" # red
PS1="\[\033[34m\]\w/\n\[\033[34m\]\u@\h[\t]\[\033[0m\]" # blue

alias rm=’rm -i’
alias cp=’cp -i’
alias mv=’mv -i’
alias ls=’ls -F –color’
alias su=’su -‘

export EDITOR=vi
export LANG=ja_JP.UTF-8
```


NIC
---
```bash
$ vi /etc/network/interfaces
allow-hotplug enp2s0
iface enp2s0 inet static
        address 192.168.xxx.xxx
        netmask 255.255.255.0
        gateway 192.168.xxx.xxx
```


ssh
---
```bash
$ apt install ssh
```


NTP
---
```bash
$ apt install ntpdate
$ crontab -e
0 3 * * * /usr/sbin/ntpdate (NTPサーバ)
```


boinc-client
------------
```bash
$ apt install boinc-client
$ boinccmd –project_attach [プロジェクトのURL] [アカウント・キー]
```


Apache
------
```bash
$ apt install apache2
$ vi /etc/apache2/apache2.conf
ServerName <サーバ名>
HostnameLookups On
$ a2dissite default
$ cd /etc/apache2/site-available/
$ cp default <サイト名>
$ vi <サイト名>
NameVirtualHost *:80 # 1つのサイトでのみ定義
<VirtualHost *:80>
  …
  <Directory /var/www/>
    …
    AuthType Basic # BASIC 認証
    AuthName "xxxx"
    AuthUserFile /var/www/.htpasswd
    Require valid-user
  </Directory>
  …
</VirtualHost>
$ a2ensite <サイト名>
$ /etc/init.d/apache2 restart
```


Apacheリバースプロキシ
----------------------
```bash
$ cd /etc/apache2/mods-available/
$ cp -p proxy.conf proxy.conf.org
$ vi proxy.conf
ProxyRequests Off
<Proxy *>
    Order deny,allow
    Allow from all
</Proxy>
$ a2enmod proxy
$ a2enmod proxy_http
$ vi /etc/apache/site-available/xxxx
ProxyPass / http://xxx.xxx.xxx.xxx/
ProxyPassReverse / http://xxx.xxx.xxx.xxx/
$ /etc/init.d/apache2 restart
```


Redis
-----
```bash
$ apt-get install redis-server redis-tools
$ vi /etc/redis/redis.conf
$ /etc/init.d/redis-server restart
```


LVM
---
|用語|意味|
|:-|:-|
|LVM(Logical Volume Manager)|ボリューム管理ソフトウェア|
|PV(Physical Volume)|物理ボリューム、物理ディスク(例:/dev/sdb)／パーティション(例:/dev/sdc1)と1:1に対応する|
|VG(Volume Group)|ボリュームグループ、1個以上の物理ボリュームを束ねたもの|
|LV(Logical Volume)|論理ボリューム、VG内から複数のLVを切り出して利用する、LV上にファイルシステムを作成しマウントする|
|エクステント|LVSにおいてVG/LVを利用する際の最小単位|

  * インストール&セットアップ
  ```bash
  $ aptitude install lvm2  # インストール
  # 物理ディスクの準備
  # 物理ディスク上のパーティションをPVとして登録する場合、fdiskなどでパーティションを作成
  $ pvcreate /dev/sdb  # PV(Physical Volume)作成
  $ vgcreate vg0 /dev/sdb  #VG(Volume Group)作成
  $ vgdisplay vg0  #  空きエクステント数の確認
  …
  Free  PE / Size       255 / 1020.00 MB
  …
  $ lvcreate -l 255 -n lv0 vg0  # VG上にLV作成
  $ mkfs.ext3 /dev/vg0/lv0  # ファイルシステム作成
  $ mount /dev/vg0/lv0 /mnt  # ファイルシステムのマウント
  ```

  * 物理ディスク追加
  ```bash
  $ pvcreate /dev/sdb  # PV(Physical Volume)作成
  $ vgextend vg0 /dev/sdc  # VG(Volume Group)拡張
  $ lvextend -l 510 /dev/vg0/lv0  # LV(Logical Volume)拡張
  $ resize2fs  /dev/vg0/lv0  # ファイルシステム拡張
  ```

  * 物理ディスク削除
  ```bash
  $ umount /mnt
  $ e2fsck -f /dev/vg0/lv0
  $ resize2fs /dev/vg0/lv0 800m  # ファイルシステム縮小、計算が面倒なので少し小さめに縮小しておく
  $ mount /dev/vg0/lv0 /mnt
  $ lvreduce -l 255 /dev/vg0/lv0  # LV縮小
  $ resize2fs /dev/vg0/lv0  # ファイルシステム拡張、LVを縮小しておいてから拡張して、LV全体を利用
  $ pvmove /dev/sdb  # PV(Physical Volume)移動、解除したいPVからデータを移動
  $ vgreduce vg0 /dev/sdb  # PV(Physical Volume)解除
  ```


sshパスワードなしログイン
-------------------------
sshでログインする元のサーバをsshクライアント側とし、ログイン先のサーバをsshサーバ側とする。

  * sshクライアント側
  ```bash
  $ ssh-keygen -t rsa
  $ cat .ssh/id_rsa.pub > authorized_keys
  ```

  * sshサーバ側
  ```bash
  $ mkdir .ssh
  $ chmod 700 .ssh
  ```

  * sshクライアント側
  ```bash
  $ scp authorized_keys (sshサーバ):/(ホームディレクトリ)/.ssh/
  ```
  ここではsshサーバ側にauthorized_keysがない前提だが、存在している場合は追記する。


php5→php7
----------
```bash
$ vi /etc/php/7.3/apache2/php.ini
$ a2enmod php7.3
$ systemctl restart apache2
```


cron-apt
--------
```bash
$ apt install cron-apt
$ vi /etc/cron-apt/action.d/3-download
autoclean -y
safe-upgrade -y -o APT::Get::Show-Upgraded=true
```
