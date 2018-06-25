# CentOSにSamba(FileServer)を構築する

先輩からなんか手に入れたので今回はcentOS7にSambaを入れてファイルサーバを構築してみようと思います。
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

`/etc/yum.repos.d/CentOS-Base.repo`

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


