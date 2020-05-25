.. post:: 22 May, 2020
   :tags: Vim, Python, Linux, FlashTool, Android
   :category: Linux
   :author: w.tknv
   :language: jp

Rooting SONY Xperia Z5 501SO
===============================

.. image:: img/may/tapir.JPG
   
ソフトバンクのZ5を他のアンドロイドイメージにしてルート権限奪取．ソフトバンクがバンドルしたアプリやお財布ケータイは消える．

イメージ書き換えにはFlashTool(この時のバージョン0.9.28.0)というのがあるようで，それをLinuxで動かすにはglibc2.29以上が必要．

ubuntu 18.04 LTSでglibc単体をアップグレードするか考えたが，どのみちシステムは最近20.04 LTS（この時glibcは2.31）がでた為，アップグレードした方が後々楽そうなのでシステムのアップグレードをした．

そしてvimが動かなくなった．libperlのバージョン違いから始まり修正しても次々とリンク先なしエラー．いずれにせよvimがないと始まらないしパイソンの3.8.3も出たようなのでそっち入れてからvimのビルド．

Vimのビルド
------------------

githubからvimのリポをクローンして．

.. code-block:: bash

    ./configure \
    --with-features=huge \
    --enable-multibyte \
    --enable-rubyinterp=yes \
    --enable-python3interp=yes \
    --with-python3-config-dir=$(python3-config --configdir) \
    --enable-luainterp \
    --enable-perlinterp=yes \
    --enable-gui=gtk2 \
    --enable-cscope \
    --prefix=/usr/local \
    --enable-fail-if-missing

| ビルド `make -j$(nproc) VIMRUNTIMEDIR=/usr/local/share/vim/vim82`
| Linuxで使っていないようなヘッダがないエラーでビルドがコケる場合は，`make reconfig`

.. code-block:: bash

   os_unixx.h:53:11: fatal error: stropts.h: No such file or directory
      53 | # include <stropts.h>

ビルド完了したら，`sudo make install` かdpkgとしてインストールするなら `sudo apt install checkinstall` の後に，vimのソース内で `sudo checkinstall`

vimのオプションがイネーブルになってるか確認 `vim --version` 

vimをメインエディタに．

.. code-block:: bash

   sudo update-alternatives --install /usr/bin/editor editor /usr/local/bin/vim 1
   sudo update-alternatives --set editor /usr/local/bin/vim
   sudo update-alternatives --install /usr/bin/vi vi /usr/local/bin/vim 1
   sudo update-alternatives --set vi /usr/local/bin/vim

ソニー　エクスペリア　Z5 ソフトバンクのルート権限奪取
--------------------------------------------------------------

先ずソフトバンクのイメージのままでルート権限奪取する方法ではないです．ドイツのファームウェアになります．必要なファームウェア，ソフトウェア，やり方はこちらを参照に．
`XDA <https://forum.xda-developers.com/xperia-z5/general/ftf-e66xx-patched-kernels-google-drive-t3688365>`_ https://forum.xda-developers.com/xperia-z5/general/ftf-e66xx-patched-kernels-google-drive-t3688365

| Linux用のFlashTool: `FlashTool <http://www.flashtool.net/downloads_linux.php>`_ http://www.flashtool.net/downloads_linux.php
| ソースコード: https://github.com/Androxyde/Flashtool

FlashToolダウンロード後に起動してもリポのシンクか何かで止まるので止まるようなら， FlashToolを止めた後に，　`~/.flashTool/devices` の中で `git clone git@github.com:Androxyde/devices.git` にてデバイスのファイルを取得． その後にFlashTool起動でうまく行くはず．

ftfファイルがFlashToolで認識されない場合（バージョン0.9.28.0では認識されなかった）はftfファイル内のMANIFEST.MFをソフトバンクのZ5に合わせる必要有り．

MANIFEST.MFをソフトバンクのZ5に合わせる
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`E6653_32.4.A.1.54_1298-3675_R1E.ftf/META-INF/MANIFEST.MF`

.. code-block:: inf

   Manifest-Version: 1.0
   cda: 1298-3675
   branding: GENERIC_32.0.C.0.380
   cmd25: false
   version: 32.4.A.1.54
   device: 501SO
   Created-By: FlashTool
   noerase: APPS_LOG,DIAG,SSD,PERSIST,USERDATA,CUST-RESET,MASTER-RESET,RE
   SET-WIPE-REASON,SIMLOCK
   revision: R1E

ブートローダーのアンロック
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

これをしないとルート権限奪取の為のイメージが焼けない．

`Sony Developer Site <https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader/#unlock-code>`_ https://developer.sony.com/develop/open-devices/get-started/unlock-bootloader/#unlock-code
から無料でアンロックコードを入手してブートローダーのアンロック．保証対象外になる模様．すでに古くて保証は切れていると思われる．