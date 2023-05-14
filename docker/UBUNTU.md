UbuntuでDockerを動かす
===

# 0. 必要パッケージのインストール
最小構成でインストールしているため、vi(vim)すら入っていない。  
パッケージ内容を更新/vimをインストールする。

```
sudo apt update
sudo apt upgrade
sudo apt install vim
```

# 1. SSHの設定

## 1-1. SSHで接続できるか確認する
Ubuntuインストール画面にて設定画面した（はず）のSSHが有効か、ホスト側から接続する。

## 1-1. ホスト側でSSHキーを準備

ネットで検索すると方法が色々あるので、調べる。
### 注意点

- パーミッション
    - ~/.ssh ... 700
    - ~./ssh/ 配下のファイル ... 600

## 1-2. ホストからVMへ、サーバーへSSH公開鍵を送信する

```
ssh-copy-id -i ~/.ssh/id_rsa.pub ユーザー名@リモートサーバーのIPアドレス
```

## 1-3. ホストのconfigファイル

ホームディレクトリの ~/.ssh 配下に「config」というファイルを作成。
```
vi ~/.ssh/config
```

編集内容  
```
Host utm-ubuntu2204-arm
  Hostname リモートサーバーのIPアドレス
  User ユーザー名
```

ユーザー名、ホスト名は適宜読み替えること。