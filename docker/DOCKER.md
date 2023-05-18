
Docker
===

このセクションではDockerを使えるようにセットアップしていく。  

参考サイト  
[リモートホストで動作しているDockerデーモンを使用する](https://kazuhira-r.hatenablog.com/entry/2020/06/30/223609)


# 1. インストール
# 1-1. 事前準備

```
sudo apt install ca-certificates curl gnupg lsb-release
```

# 1-2. Dockerリポジトリの登録

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

# 1-3. インストール

```
sudo apt install docker-ce \
docker-ce-cli \
containerd.io \
docker-compose-plugin
```

## 1-4. 起動

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

表示結果  
```
$ sudo systemctl status docker
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2023-05-15 13:19:24 UTC; 2min 6s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 28031 (dockerd)
      Tasks: 7
     Memory: 23.4M
        CPU: 111ms
     CGroup: /system.slice/docker.service
             └─28031 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

May 15 13:19:24 utm-ubuntu2204-arm2 systemd[1]: Starting Docker Application Container Engine...
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.112812944Z" level=info msg="Starting up"
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.113276825Z" level=info msg="detected 127.0.0.53 nameserver, >
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.152007763Z" level=info msg="Loading containers: start."
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.257484641Z" level=info msg="Loading containers: done."
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.268268494Z" level=info msg="Docker daemon" commit=9dbdbd4 gr>
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.268356579Z" level=info msg="Daemon has completed initializat>
May 15 13:19:24 utm-ubuntu2204-arm2 dockerd[28031]: time="2023-05-15T13:19:24.281934011Z" level=info msg="API listen on /run/docker.sock"
May 15 13:19:24 utm-ubuntu2204-arm2 systemd[1]: Started Docker Application Container Engine.
```

# 2. dockerを一般ユーザーでも実行できるようにする

## 2-1. グループ確認

以下のコマンドを実行し、`docker`グループが存在しているか確認する。

```
cat /etc/group | grep docker
```

実行結果  

```
$ cat /etc/group | grep docker
docker:x:999:
```

ここでは既に存在していたが、存在しない場合は作成コマンドを実行する。  

```
sudo groupadd docker
```

## 2-2. ユーザーをグループに追加する

```
sudo usermod -aG docker ユーザー名
```

## 2-3. 確認

いったんログインしなおし、以下を実行してみる。

```
docker run hello-world
```

実行結果  
```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
70f5ac315c5a: Pull complete
Digest: sha256:fc6cf906cbfa013e80938cdf0bb199fbdbb86d6e3e013783e5a766f50f5dbce0
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
 ```

# 3. 2375ポートをLISTENさせる

## 3-1. 一旦停止

以下のコマンドで停止を試みると、ワーニングが表示されて停止できない。  
docker.socket がまだ残っているから停止できない。  

```
# 試した停止コマンド
$ sudo systemctl stop docker.service
Warning: Stopping docker.service, but it can still be activated by:
  docker.socket

```

docker.socket を停止させる。

```
sudo systemctl stop docker.socket
```

## 3-2. VM内のdockerの設定変更

VM側のUbuntuにて、`/etc/systemd/system/multi-user.target.wants/docker.service` を編集する。

```
# 編集する
$ sudo vi /etc/systemd/system/multi-user.target.wants/docker.service
```

```
# 変更前
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock
```

```
# 変更後
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H fd:// --containerd=/run/containerd/
```

編集が終わればサービスを再起動させる。  

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

ポートが開いていることを確認。  

```
$ ss -tnl | grep 2375
LISTEN 0      4096               *:2375            *:*
```

## 3-3. コンテキスト設定

VM内のdockerとアクセスする側には`docker-cli`がインストールしているハズである。  
`コンテキスト`と呼ばれるものを作成し、VM内のdockerと通信する。


```
# 現在のコンテキストの確認
$ docker context ls
NAME                   DESCRIPTION                               DOCKER  ENDPOINT               ERROR
default                Current DOCKER_HOST based configuration   unix:///var/run/docker.sock

# 「utm-ubuntu2204-arm2」というコンテキストを作成する
$ docker context create utm-ubuntu2204-arm2 --docker host=tcp://192.168.64.3:2375
utm-ubuntu2204-arm2
Successfully created context "utm-ubuntu2204-arm2"

# 確認
$ docker context ls
NAME                   DESCRIPTION                               DOCKER ENDPOINT               ERROR
default *              Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
utm-ubuntu2204-arm2                                              tcp://192.168.64.3:2375
```

```
# コンテキストを変更
$ docker context use utm-ubuntu2204-arm2
utm-ubuntu2204-arm2
Current context is now "utm-ubuntu2204-arm2"

# 切り替わったか確認
$ docker context ls
NAME                    DESCRIPTION                               DOCKER ENDPOINT               ERROR
default                 Current DOCKER_HOST based configuration   unix:///var/run/docker.sock
utm-ubuntu2204-arm2 *                                             tcp://192.168.64.3:2375
```

## 3-4. 実行してみる

```
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (arm64v8)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
