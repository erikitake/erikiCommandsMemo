# LinuxCommands

### Pythonバージョン変更
```
cd /usr/local/bin
sudo ln -sf /usr/local/python/bin/pip3.6 pip
sudo ln -sf /usr/local/python/bin/pip3.8 pip
```
```
cd /usr/local/python/bin
sudo ln -sf /usr/local/python/bin/python3.6 python3
sudo ln -sf /usr/local/python/bin/python3.8 python3
```

### Git

- 初期設定
```
git config user.email
git config user.name
```

- コマンド例
```
organisation="password_app"
baseBranch="main"
branch="develop"
yymmdd="210625"
ver="1"

cd ~/developers/$organisation; if [ $? != 0 ]; then mkdir ~/developers/$organisation;cd ~/developers/$organisation;fi
git clone https://github.com/erikitake/$organisation -b $baseBranch $branch-$yymmdd-$ver

cd $branch-$yymmdd-$ver; git status; git branch
git checkout -b $branch; git status; git branch

#after develop
git diff
git status; git add .; git status
git diff --cached
#git add を取り消したい場合は、git reset HEAD

git commit -m "for readme"
git commit -m "initial commit"
git commit -m "log output for cloudwatch logs"
git commit -m "change subnet"
git push --set-upstream origin $branch
git push

git add . ; git commit -m "circleci" ; git push
```

### ruby
- rails起動
```
nohup bin/rails s -e production &
bin/rails s -e production
```

### Docker操作
dockerは、色々なイメージがDocker Hubに保存されていて、それを利用するイメージ。  
Dockerfileとは規定の文法で記載されたDocker Imageの作成が記録されたファイル
- Docker Hubからダウンロードしたイメージの確認  
`docker images`
- Dockerイメージの削除（コンテナが利用しているイメージは削除できない）  
`docker rmi コンテナID`
- 実行中のdockerコンテナの一覧が表示される。（-aで停止中のコンテナも表示）  
`docker ps`
- dockerイメージの停止  
`docker stop コンテナID`
- dockerコンテナの起動  
`docker restart コンテナID`

- 下記はdocker-composeで代用するため実際には使用しない  
```
##Dockerfileからイメージをビルド
##（Dockerfileのフォルダ直下で）
#docker build . -t 任意のレポジトリ名
#
##Dockerイメージをコンテナとして起動する
#dcker run -d イメージのリポジトリ名
######
```

### docker-compose操作
docker-compose は複数のコンテナを組み合わせて１つのアプリケーションとして構成を定義するファイル  
docker-compose.ymlファイルにコンテナの設定を記載している。  
このファイルを使用して一気にコンテナの起動までができる。  
このファイルにてDockerfileも参照している。  

`docker-compose up -d`  # コンテナの一斉起動（イメージも作成してくれる）  
`docker-compose down`  # コンテナの停止  
`docker-compose build` # サービスの Dockerfile やビルドディレクトリの内容を変更する場合は、docker-compose buildを実行して再ビルド  
`docker-compose up --build` # Dockerfileやdocker-compose.ymlの変更を反映、railsサーバーを再起動  
`docker-compose run web bundle install` # bundle installなどのコマンドを実行したい  

- コンテナの強制削除  
```
docker rm  -f `docker ps -a | awk 'NR>1{print $1}'`
```

- イメージの強制削除
```
docker rmi -f `docker images | awk 'NR>1{print $3}'`
```

- コンテナとイメージを一気に削除
```
docker rm  -f `docker ps -a | awk 'NR>1{print $1}'` ; docker rmi -f `docker images | awk 'NR>1{print $3}'`
```

## Linuxコマンド関連
- ユーザの追加  
```
useradd ユーザ名 -mでホームディレクトリ作成、-Gで所属グループ設定
sudo useradd -m rundocker -G docker
```

- ボリューム操作
    - 確認
        ```
        df -hT
        sudo blkid
        sudo fdisk -l
        ```

    - 物理ボリュームの確認
        ```
        sudo pvs
        sudo vgs
        sudo lvs
        ```

- Linux EBSの縮小  
    - 下記コンソール操作  
        ```
        #既存EC2の停止、コピーしたいEBSを既存EC2からデタッチ
        #新規EC2の作成
        #縮小したいサイズのEBSを作成
        #新規EC2に作成したEBS、デタッチしたEBSをアタッチ
        ```
    - 新規EC2にSSHログインし下記操作を実行  
        ```
        状態の確認
        df -hT         #マウント状態
        sudo lsblk     #マウント状態
        sudo blkid     #アタッチされているディスク
        #データコピー用フォルダ作成
        sudo mkdir -p /mnt/new /mnt/old
        #古いEBSをマウント（UUIDが重複している場合を考慮し、-o nouuidを使用。ファイルシステムは状態の確認コマンドで確認する。）
        sudo mount -t xfs -o nouuid /dev/xvdg1 /mnt/old
        #新EBSにXFSファイルシステムを作成（ファイルシステムは状態の確認コマンドで確認する。）
        sudo fdisk /dev/xvdf
        >p >n >p >1 >空Enter >空Enter >p >w
        #作成の確認
        sudo lsblk     #マウント状態
        sudo blkid     #アタッチされているディスク
        #フォーマット
        sudo mkfs.xfs /dev/xvdf1
        #作成の確認
        sudo lsblk     #マウント状態
        sudo blkid     #アタッチされているディスク
        #新しいEBSをマウント（UUIDが重複している場合を考慮し、-o nouuidを使用。ファイルシステムは状態の確認コマンドで確認する。）
        sudo mount -t xfs -o nouuid /dev/xvdf1 /mnt/new
        sudo lsblk     #マウント状態

        cd /mnt/old/
        ( LANG=C date; sudo nohup xfsdump -J - /mnt/old | sudo xfsrestore -J -p 60 - /mnt/new ; date ) > /tmp/disk-sync.log 2>&1  < /dev/null &
        df -hT
        sudo  cp -p /tmp/disk-sync.log /mnt/new/var/log/
        sudo mount -t proc /proc /mnt/new/proc
        sudo mount -t sysfs /sys /mnt/new/sys
        sudo mount --bind /dev /mnt/new/dev
        sudo chroot /mnt/new
        ls -l /var/log/disk-sync.log
        blkid
        cp -p /etc/fstab{,_$(date +'%Y%m%d')}; ls -l /etc/fstab*
        vi /etc/fstab
        cp -p /boot/grub2/grub.cfg{,_$(date +'%Y%m%d')}; ls -l /boot/grub2/grub.cfg*
        grub2-mkconfig -o /boot/grub2/grub.cfg
        grub2-install /dev/xvdf
        sudo umount /mnt/old/ /mnt/new/proc/ /mnt/new/sys/ /mnt/new/dev/; sudo umount /mnt/new/
        df -hT
        ```

- ボリューム同期  
```
sudo rsync -ax /mnt/old /mnt/new
```
- アンマウント
```
sudo umount /mnt/new /mnt/old
```
- フォルダ削除
```
sudo rmdir /mnt/new /mnt/old
```

```
下記コンソール操作
#既存EC2の停止、すべてのEBSを既存EC2からデタッチ
#必要なEBSをアタッチ（ルートデバイス(EC2コンソール参照)を指定すること！(例：/dev/xvda))
#EC2起動

#物理ボリュームの拡張
sudo pvresize /dev/sda5

#論理ボリュームの拡張
sudo lvextend -l +100%FREE /dev/mapper/rhel-home

#ファイルシステムの拡張
sudo xfs_growfs /home
```
AWS EC2でボリュームの拡張をする場合は、AWSの記事にコマンドが記載されているので参照


- ファイル転送
```
scp -i 証明書ファイル 転送したいファイル userName@IPorDomain:/path/
scp -i 証明書ファイル userName@IPorDomain:/path/転送したいファイル ./
```

- 圧縮・解凍コマンド
- 圧縮
```
tar cvfz ファイル名.tar.gz 配置先ディレクトリ
```
- 解凍
```
tar xvfz 圧縮ファイル名.tar.gz
```
- シェル変更  
- zshに変更する場合
```
sudo chsh -s /bin/zsh ユーザ名
sudo su - ユーザ名
vi .zshrc
```

- パスワード設定
```
sudo passwd ユーザ名
```

### サーバ起動時にサービス起動させる設定
- リスト表示
```
sudo systemctl list-unit-files -t service
```
#enabledになっていない場合システムスタート時に起動されない

- 起動設定
```
sudo systemctl enable nginx.service
```
- 解除
```
sudo systemctl disable nginx.service
```
- 状態確認
```
sudo systemctl status nginx.service
```

- 設定ファイル
```
cd /usr/lib/systemd/system
sudo vi unicorn.service
```
下記ファイル内容
```
[Unit]
Desctiption=unicorn start

[Service]
User=dapp
WorkingDirectory=/var/www/dapp/production/current
SyslogIdentifier=unicorn.service
PIDFile=/var/www/dapp/production/current/tmp/pids/unicron.pid
Environment='RILS_ENV=production'
ExecStart=/home/dapp/.rbenv/bin/rbenv exec bundle exec "unicorn_rails -c /var/www/dapp/pruduction/current/config/unicorn/prudction.rb -E production -D"
ExecStop=/usr/bin/kill -QUIT $MAINPID
EsecReload=/bin/kill -USR2 $MAINPID

[Install]
WantedBy=multi-user.target
```

### SELinux無効化
```
getenforce
sudo setenforce 0;getenforce
#Enforcing 有効
#Permissive 無効
```

### pumaサービス化
```
sudo systemctl daemon-reload
sudo systemctl start puma
systemctl status puma
```

```
journalctl -xe
sudo vi /usr/lib/systemd/sysmte/puma.service
```

### LinuxサーバのDNS設定
```
sudo nmcli connection
sudo nmcli connection show XXX
aaa=eth0 ; echo $aaa
sudo nmcli connection modify $aaa
ipv4 dnx "IP,IP,IP"; sudo nmcli connection show $aaa
sudo systemctl restart network.service
```

### LinuxサーバのNTP設定
chronyc sources
sudo vi /etc/chrony.conf
IP
sudo sytemctl restart chronyd; sleep 1; sudo sytemctl status chronyd
chronyc sources

### SSL証明書関連
証明書の確認  
```
openssl s_client -connect <IP:443> -showcerts
```

有効期限  
```
openssl s_client -connect <IP:443> | openssl x509 -noout -enddate 
openssl x509 -nooout -text -in /etc/nginx/conf.d/gagapi.pem
```

### sed awk xargs
sed -ne '2,$p' -e 's/  */ /g' | awk '{print $1":"$2}'`
#docker images -f 'dangling=true' --format '' | xargs docker rmi


