UbuntuでDockerを動かす
===

# 1. OSアップデート
最小構成でインストールしているため、vi(vim)すら入っていない。  
パッケージ内容を更新/vimをインストールする。

```
sudo apt update
sudo apt upgrade
sudo apt install vim
```

## 1-2. apt install すると毎回「Daemons using outdated libraries」と表示される

```
Running kernel seems to be up-to-date.

Restarting services...
Daemons using outdated libraries
--------------------------------

  1. dbus.service        3. networkd-dispatcher.service  5. polkit.service          7. unattended-upgrades.service  9. none of the above
  2. multipathd.service  4. packagekit.service           6. systemd-logind.service  8. user@1000.service

(Enter the items or ranges you want to select, separated by spaces.)

Which services should be restarted?
```

毎回問い合わせになるので、確認なしに再起動してほしいので、以下の設定を行う。  
/etc/needrestart/conf.d/ に　`50local.conf` というファイルを作成する。

```
$ echo "\$nrconf{restart} = 'a';" | sudo tee /etc/needrestart/conf.d/50local.conf
```

参考サイト  
[【2022年版】 Ubuntu 22.04 で apt install すると、Which services should be restarted? ときかれる | Qiita](https://qiita.com/nouernet/items/ffe0615c14147863de7a)


# 2. SSHの設定

## 2-1. SSHで接続できるか確認する
Ubuntuインストール画面にて設定画面した（はず）のSSHが有効か、ホスト側から接続する。

## 2-2. ホスト側でSSHキーを準備

ネットで検索すると方法が色々あるので、調べる。
### 注意点

- パーミッション
    - ~/.ssh ... 700
    - ~./ssh/ 配下のファイル ... 600

## 2-3. ホストからVMへ、サーバーへSSH公開鍵を送信する

```
ssh-copy-id -i ~/.ssh/id_rsa.pub ユーザー名@リモートサーバーのIPアドレス
```

実行結果  
```
ssh-copy-id -i ~/.ssh/id_rsa.pub ■■■@192.168.64.3
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/Users/■■■/.ssh/id_rsa.pub"
The authenticity of host '192.168.64.3 (192.168.64.3)' can't be established.
ED25519 key fingerprint is SHA256:B3GBa/pqLBch+OtXmKFo4V4oD7ahubBElcNn9YvMcE8.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
■■■@192.168.64.3's password:

Number of key(s) added:        1

Now try logging into the machine, with:   "ssh '■■■@192.168.64.3'"
and check to make sure that only the key(s) you wanted were added.
```


## 2-4. ホストのconfigファイル

ホームディレクトリの ~/.ssh 配下に「config」というファイルを作成。
```
vi ~/.ssh/config
```

編集内容  
```
Host utm-ubuntu2204-arm
  Hostname リモートサーバーのIPアドレス
  IdentityFile ~/.ssh/id_rsa
  User ユーザー名
```

ユーザー名、ホスト名は適宜読み替えること。


## 2-5.　sshからのrootユーザーのログインを禁止する。

設定ファイルに　`PermitRootLogin no` を追記する。  
```
sudo vi /etc/ssh/sshd_config
```


# 3. Docker インストール

## 3-1. 事前準備

```
sudo apt install ca-certificates curl gnupg lsb-release
```

## 3-2. Dockerリポジトリの登録

```
$ sudo mkdir -p /etc/apt/keyrings
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt update
```

解説  
1行目：curl コマンドで Docker の Ubuntu用GPG鍵をダウンロード。  
2行目：標準のOpenGPGエンコーディングに変換し、aptのキーリングディレクトリへ保存。  
3行目：パーミッション変更。  
4行目：Docker アーカイブをダウンロード。  
5行目：aptパッケージ一覧情報の更新。  

## 3-3. インストール

```
sudo apt install docker-ce \
docker-ce-cli \
containerd.io \
docker-compose-plugin
```

## 3-4. 起動

起動コマンド  
```
sudo systemctl start docker
```

自動起動設定  
```
sudo systemctl enable docker
```

ステータス表示  
```
sudo systemctl status docker
```


