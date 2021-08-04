# EC2LinuxEnvSetCmds
## EC2 Linuxをたてたらまずやること、環境設定

### yum インストール
`sudo yum install -y git postgresql.x86_64`  
`sudo yum install -y libxml2 libxslt`   #pythonのlxmlで必要だったか?

### gitの設定
`git config --global user.name erikitake`  
`git config --global user.email e.rikitake@icloud.com`  
push時の認証省略  
`vi ~/.netrc`  
```
machine github.com
login erikitake
password enter your password
```

### Pythonのインストール
- まずインストールしたいバージョンの指定  
`export PYVER=3.6.0`
- インストール  
    ```
    sudo yum install -y make gcc zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel libffi-devel
    cd ; wget https://www.python.org/ftp/python/$PYVER/Python-$PYVER.tgz
    tar xvzf Python-$PYVER.tgz; cd Python-$PYVER
    ./configure --prefix=/usr/local/python
    make
    sudo make install
    ```

- 環境設定とバージョン切り替え
    ```
    vi ~/.bash_profile
    source ~/.bash_profile
    sudo ln -s /usr/local/python/bin/python3 /usr/local/bin/python
    sudo ln -s /usr/local/python/bin/pip3.8 /usr/local/bin/pip
    python -V
    pip -V
    ```
もし再インストールしたい場合は、そのままconfigure,make,make installすればよい。（削除は、sudo make uninstall）

### nodejsのインストール
https://docs.aws.amazon.com/ja_jp/sdk-for-javascript/v2/developer-guide/setting-up-node-on-ec2-instance.html

### SAMのインストール
https://docs.aws.amazon.com/ja_jp/serverless-application-model/latest/developerguide/serverless-sam-cli-install-linux.html

<メモ>
- dockerインストール後にサービスの起動設定  
    `sudo systemctl enable docker`  
    なぜかユーザ権限が反映されないので、システム再起動  
    `sudo reboot`  
    docker ps ができれば次、webの手順に戻ってwgetでダウンロード  
    wget ・・・・

