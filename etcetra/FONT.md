自分が使うフォントを整理する
===

# Ricty with Powerline
シェルでも使う。  
最後の2行のコマンドは、 brew からインストールが完了した後にも表示されるので、そちらを確認したほうがいい。  

```bash
$ brew tap sanemat/font
$ brew install ricty --with-powerline
$ cp -f /usr/local/opt/ricty/share/fonts/Ricty*.ttf ~/Library/Fonts/
$ fc-cache -vf
```

# MigMix
コーディングするときは等幅を使う。

[公式サイト](https://mix-mplus-ipa.osdn.jp/migmix/)

- Zipファイルを解凍したttfファイルを右クリックからインストール。
