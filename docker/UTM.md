UTMでLinux VMを構築する
===

# 1. UTM をインストール

- App Store 版は有料。
- [サイト](https://getutm.app/) からダウンロード。(サポートなし)

# 2. ARM版Ubuntu Serverをダウンロード
[ARM版のUbuntu Server](https://ubuntu.com/download/server/arm) の ISO イメージをダウンロード。  
今回は22.04.02 LTS を取得。

# 3. UTMでVMを作成
むちゃくちゃ簡単。

## 3-1. 「+」ボタンをクリック

![001.png](img/build-docker-env-utm/001.png)

## 3-2. 「仮想化」をクリック

![002.png](img/build-docker-env-utm/002.png)

## 3-3. 「Linux」をクリック

![003.png](img/build-docker-env-utm/003.png)

## 3-4. 仮想化エンジンの設定
- 「Apple仮想化を使用」はチェック入れない。
-  「カーネルイメージから起動」はチェックいれない。
- 「起動ISOイメージ」の選択ボタンをクリック。

![004.png](img/build-docker-env-utm/004.png)

## 3-5. ハードウェア
- メモリ、CPUコアは、自分の端末に適した量を指定する。
- 今回はDocker用途なので、OpenGLは使用しない。

![005.png](img/build-docker-env-utm/005.png)

## 3-6. ストレージ
- ここも、自分の端末に適した量を指定する。
- 30GBもあれば充分？

![006.png](img/build-docker-env-utm/006.png)

## 3-7. 共有ディレクトリ

開発に必要なソースファイルはホスト側に置くので、共有ディレクトリとして指定しておく。  
※VM側で有効にするには /etc/fstab への変更が必要。  

![007.png](img/build-docker-env-utm/007.png)

## 3-8. 概要

先程まで設定した内容の確認画面が表示される。  
名称はわかりやすくしたほうがいい。

![008.png](img/build-docker-env-utm/008.png)

## 3-9. Dockerを利用する準備

<span style="font-size:200%; COLOR: red;">!!結構重要!!</span>だったりする。  
このままの状態でUbuntuをインストールすると、IPアドレスは192.168.*.*のアドレスが付帯される。  
これはネットワークデバイスのデフォルトが「共有ネットワーク」になっているためである。  

Docker Desktop から起動したコンテナへのアクセスは「localhost:ポート番号」というIPアドレス（と、ポート番号）になるので、UTMを使ってもこの通りにしたい。  

### 3-9-1. VM設定を編集

UTMのVM一覧にて該当VMを選択し、右クリックメニュー「編集」を選択。  
![035.png](img/build-docker-env-utm/035.png)

### 3-9-2. 既存のネットワークデバイスを削除

左ペインにある「デバイス」欄の中にある「ネットワーク」を選択し、右クリックメニュー「削除」をクリック。  
![036.png](img/build-docker-env-utm/036.png)

### 3-9-3. デバイス追加

続いて「デバイス」欄の「＋新規...」をクリックすると、追加したいデバイスがプルダウンされるので、中から「ネットワーク」を選択。  
![037.png](img/build-docker-env-utm/037.png)

↓↓  
追加した「ネットワーク」を選択し、ネットワークモードを「仮想VLAN」に変更する。  
SSH、Portaine(後記)で使用するポートの転送が必要のため、「ポート」をクリック。  
![038.png](img/build-docker-env-utm/038.png)

↓↓  
ゲストポートとホストポートを設定する。  
![039.png](img/build-docker-env-utm/039.png)

| 用途 | ゲストポート | ホストポート | 備考 |
| --- | --- | --- | --- |
| SSH | 22 | 23 | 外部からホストにSSH接続しなければ「22」でいいかも |
| Docker | 2375 | 2375 | ホストのdocker-cliからコンテキスト接続する |
| Portainer | 9000 | 9000 | 自己署名状態にしないので9000 |

※コンテナを追加したり、VM内へ通信したいときはポートフォワード設定を行う。  
![040.png](img/build-docker-env-utm/040.png)


# 4. Ubuntuのインストール

## 4-1. 
作成したVMを選択し、起動ボタンをクリック。
![009.png](img/build-docker-env-utm/009.png)

## 4-2. 
`Try or Install Ubuntu Server`を選択。
![010.png](img/build-docker-env-utm/010.png)

言語は「English」を選択。日本語はない。
![011.png](img/build-docker-env-utm/011.png)

キーボードレイアウトは、自分が使っているキーボードを指定し、「Done」で次の設定へすすむ。  
(ここには「Japanese」がある)  
![012.png](img/build-docker-env-utm/012.png)

## 4-3. 
インストールタイプを選択。
Dockerデーモンを実行するための最小構成で構築したいので、「Ubuntu Server (minimized)」を選択し「Done」を選択。

![013.png](img/build-docker-env-utm/013.png)


## 4-4. 
ネットワークはそのままではDHCPによるIPアドレス指定になるため、固定にする。  
ネットワークカード名のところまでカーソルを移動しenterキーで選択.  
![014.png](img/build-docker-env-utm/014.png)

「Edit IPv4」を選択。
![015.png](img/build-docker-env-utm/015.png)

「IPv4 Method」が「Automatic (DHCP)」となっているので、enterキー押下から「Manual」を選択。  
![016.png](img/build-docker-env-utm/016.png)
![017.png](img/build-docker-env-utm/017.png)

IPアドレスを設定。  
今回セットアップしたVMは「192.168.64.3」だったので、それを踏襲。  
DHCP から static に変わっている。    
![018.png](img/build-docker-env-utm/018.png)  
![019.png](img/build-docker-env-utm/019.png)

プロキシサーバーの設定が必要ない場合は、そのまま「Done」。  
![020.png](img/build-docker-env-utm/020.png)

ミラーサーバーは、自動的に近くのサーバーが指定される。  
![021.png](img/build-docker-env-utm/021.png)

## 4-5.
ストレージのパーティーションはデフォルト。  
「Done」ですすめるとレイアウトが表示される。
![022.png](img/build-docker-env-utm/022.png)   
![023.png](img/build-docker-env-utm/023.png) 

## 4-6. 
最終確認ダイアログが表示されるので「Continue」を選択。  
![024.png](img/build-docker-env-utm/024.png) 

## 4-7.
ユーザープロフィール設定。  
コンソールなどで表示されるユーザー名は、３項目名のところ。
![025.png](img/build-docker-env-utm/025.png) 

## 4-8.
「Ubuntu Pro」へのアップグレードを勧められるが、スキップ（赤枠）を選択。  
![026.png](img/build-docker-env-utm/026.png) 

## 4-9.
Dockerはコンソールで操作する予定なので、SSHコンソールができるようにする。  
![027.png](img/build-docker-env-utm/027.png) 

## 4-10.
インストールするサーバーパッケージの選択。  
今回はUbuntuになれるため、Docker は敢えてコマンドからインストールしたいので、ここでは無選択とする。  
![028.png](img/build-docker-env-utm/028.png) 

## 4-11.
インストール中は記号がクルクル周り、インストールが終わると「Reboot」が表示されるので、再起動を行う。  
![029.png](img/build-docker-env-utm/029.png) 
![030.png](img/build-docker-env-utm/030.png) 

## 4-12.
真っ黒な画面になるので、一旦ウィンドウを閉じる。  
![031.png](img/build-docker-env-utm/031.png)  
![032.png](img/build-docker-env-utm/032.png) 

## 4-13.
作成したVMにはまだISOイメージファイルがセットされているので、開放(消去)し、VMを再起動する。  
![033.png](img/build-docker-env-utm/033.png) 


# 5. 再起動後
再起動後、ログインプロンプトが表示されるので、4-4. で設定したユーザー名とパスワードを入力してログインする。
![034.png](img/build-docker-env-utm/034.png) 

つづきは、[こちら](./UBUNTU.md)でUbuntuの設定を行っていく。