# CentOSにSamba(FileServer)を構築する

知り合いから「もういらない」と言っていたmousecomputerを手に入れたので今回はcentOS7にSambaを入れてファイルサーバを構築してみようと思います。
今後はVPNを使って外部からでもファイルの操作ができるようにしていこうと思ってます。

## 動作環境
- MouseComputer desktop  
- CentOS7


## 1.CentOSのイメージファイルを取得  
CentOS7のイメージを作成[CentOS公式サイト](https://www.centos.org/download/)からイメージをダウンロードしてから、DVD-Rに落とす。焼くともいうのかな？

Image

## 2.rootでログイン
初期設定で設定したパスワードを使って、rootでログインする。

## 3.ログインした後にyumのアップデートをする。　　
以下のコマンドでyumのアップデートをしてください。

`[...]#yum update`

もし、できないまたはエラーが発生した場合、CentOS-Base.repoファイルを以下の通りに書き加えてください。

`[...]#vim /etc/yum.repos.d/CentOS-Base.repo`

[base]
name=CentOS-$releasever - Base
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=os&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
priority=1 <--★ 追記する

#released updates
[updates]
name=CentOS-$releasever - Updates
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=updates&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/updates/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
priority=1 <--★ 追記する

#additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
mirrorlist=http://mirrorlist.centos.org/?release=$releasever&arch=$basearch&repo=extras&infra=$infra
#baseurl=http://mirror.centos.org/centos/$releasever/extras/$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7
priority=1 <--★ 追記する

参照(https://tech.hitsug.net/?CentOS-7%2Fyum)


## 4.sambaのインストールと設定
- sambaのインストール  
     `[...]#yum -y install samba samba-client samba-winbind samba-winbind-client`  
     前から順にsamba本体、sambaのクライアント用コマンド、samba機能ライブラリ、samba機能ライブラリコマンドのインストール

- firewallの設定  
     `[...]#firewall-cmd --add-service=samba --permanent`   ##sambaへのアクセス許可をする  
     `[...]#firewall-cmd --add-service=samba-clilent permanent`   ##sambaクライアントのアクセス許可をする  

- SELinuxの設定  
     `[...]#getsebool -a | grep samba`  ##SELinuxの設定の確認  
      
     `[...]#setsebool -P samba_create_home_dirs on`  
     `[...]#setsebool -P samba_domain_controller on`  
     `[...]#setsebool -P samba_enable_home_dirs on`  
     `[...]#setsebool -P samba_export_share_all_rw on`  
     `[...]#setsebool -P samba_share_nfs on`  
     `[...]#setsebool -P tmpreaper_use_samba on`  
     `[...]#setsebool -P use_samba_home_dirs on`  
      
     設定が終わったら一応確認しておこう。'on'意外にも確認したいなら後ろの'| grep 'on''をなしでコマンドを打てば、sambaに関するSELiunxのすべての設定が出力されます。  
     `[...]# getsebool -a I grep samba I grep 'on'`  ##先ほど設定したSELiunxの'on'のみを表示

- 自動起動の設定
     `[...]#systemctl enable smb.service`  ##smbの自動起動の有効化  
     `[...]#systemctl enable nmb.service`  ##nmb自動起動の有効化  
     `[...]#systemctl enable winbind.service`  ##winbindの自動起動の有効化  

- ワークグループの設定  
Windowsと同じワークグループに属するようにsmb.confの設定内容を変更します。  
     `[...]#vim /etc/samba/smb.conf`

   smb.conf  
     (途中省略)  
          [global]  
               wrokgroup = WORKGROUP  ##デフォルトではMYGROUPとなっているはず？なのでWROKGROUPに書き換える  
     (以後省略)  

- 起動
        とりあえずここまでで細かい設定は終わりなので各サービスを起動  
        `[...]#systemctl start smb.service`  ##smbの起動  
        `[...]#systemctl start nmb.service`  ##nmbの起動  
        `[...]#systemctl start winbind.service`  ##winbindの起動  


## 5.アカウント管理  
- Sambaアカウントを追加
     `[...]#useradd (作成したいユーザ名)`  ##Centos7にユーザを追加  
     `[...]#passwd (作成したユーザ名)`  ##作成したユーザにパスワードを追加  
     `[...]#pdbedit -a (作成したユーザ名)`  ##pdbeditコマンドを使ってSambaアカウント追加  
     `new password:`  
     `retype new password:`  
     `Unix username: (作成したユーザ名)`  
     `(以降省略)`

     Centosにアカウントを追加するのは'useradd'、Sambaにアカウントを追加するのは'pdbedit'コマンドを使う

- Sambaアカウントの一覧  
     `[...]#pdbedit -L`  
     `(作成したユーザ名):1001:`  ##数字は作成していくと変わる

- Sambaアカウントのパスワードを変更  
     `[...]#smbpasswd (ユーザ名)`  
     `New SMB password:`  ##入力しても何も表示ません  
     `Retype new password:`  ##上と同じく何も表示されません  

- Sambaアカウントの削除  
     `[...]#pdbedit -x (ユーザ名)`  ##Sambaアカウント消去  
     `[...]#pdbedit -L`  ##本当に削除できているか確認  

- Sambaグループの追加でユーザをまとめて管理  
     `[...]#groupadd (グループ名)`  
     `[...]#usermod -aG (ユーザを追加したいグループ名) (追加するユーザ名)`  

     他にもコマンドがあるので調べて見るのもいいかも
     - https://eng-entrance.com/linux-command-usermod
     - https://eng-entrance.com/linux-command-groupmod#groupadd

     どんなgroupを作成したか忘れてしまった場合は、groupの一覧が/etc/groupに保存されているので確認してみてください。

- これから使うフォルダを作成  
     `[...]#mkdir /(作成するフォルダの名前 個人的にオススメは/home/(フォルダ名))`  
     `[...]#chgrp (所有するグループ名またはユーザ名) (上記で作成したフォルダのパス)`  
     `[...]#restorecon (上記で作成したフォルダのパス)`  

     restoreconコマンドは説明ができないので各々でググってw  
     一応参考になりそうなのを見つけたので見てみてください。  
     - https://access.redhat.com/documentation/ja-jp/red_hat_enterprise_linux/6/html/security-enhanced_linux/sect-security-enhanced_linux-working_with_selinux-selinux_contexts_labeling_files





