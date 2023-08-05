+++
title = "Pythonでnetcatを実装する#1"
date = "2023-8-5"
description = "コマンドラインからデータを送受信できるnetcatをpythonを使って実装します。"
draft = false
header_img = ""
toc = true
tags = ["python","network"]
comment = true
+++

## 1. はじめに
### デモ
## 2. 準備
### 2.1 事前知識
#### netcat
コマンドラインからTCPやUDPを使ってデータを送受信できるツールです。
利用例としては、宛先のIPアドレスとポート番号を指定して対象のホストに接続することでクライアントとして利用したり、自身のポート番号を指定して接続を待ち受けることでサーバーとして利用したりします。

#### socket

### 2.2 環境 
## 3. 実装
### 3.1 クライアントサーバー機能
TCPとUDPを使ってクライアントサーバー機能を実装します。  
まずはじめにコマンドライン引数の処理と関数の実行のための`__main__`ブロックを作ります。  
```python:client-server.py
import argparse

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('-l', '--listen', action='store_true')
    parser.add_argument('-t', '--target')
    parser.add_argument('-p', '--port', type=int)
    parser.add_argument('-u', '--udp', action='store_true')
    args = parser.parse_args()

    nc = NetCat(args)
    nc.run()
```
次に`NetCat`クラスを作成していきます。  
コマンドライン引数からクライアントとサーバーのどちらで起動するかを設定します。
```python:client-server.py
class NetCat:
    def __init__(self, args):
        self.args = args

    def run(self):
        # クライアント or サーバー
        if self.args.listen:
            # サーバー
            self.listen()
        else:
            # クライアント
            self.send()

    # サーバーの処理
    def listen(self):
        pass
    
    # クライアントの処理
    def send(self):
        pass
```

次にサーバー側の処理を書いていきます。  
サーバーとして待ち受けるプロトコルが`TCP`なのか`UDP`なのかで処理を分けます。
```python:client-server.py
    # サーバーの処理
    def listen(self):
        # UDPの処理
        if self.args.udp:
            # ソケットを作成（IP4, UDP）
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            # ソケットをバインド
            self.socket.bind((self.args.target, self.args.port))
            print('UDP Listening')
            # 待ち受ける
            while True:
                try:
                    # データを受信
                    msg, client_address = self.socket.recvfrom(1024)
                    print(f"message: {msg.decode('utf-8')}  from: {client_address}")
                    # データを返信
                    self.socket.sendto('Success to receive message'.encode('utf-8'), client_address)
                except KeyboardInterrupt:
                    self.socket.close()
                    exit()
        # TCPの処理
        else:
            # ソケットを作成（IP4, TCP）
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            # ソケットをバインド
            self.socket.bind((self.args.target, self.args.port))
            self.socket.listen()
            print('TCP Listening')
            # 接続されるまで待ち受け
            client_socket, client_address = self.socket.accept()
            print(f"Connection with {client_address} !")
            while True:
                try:
                    # データを受信
                    msg = client_socket.recv(1024)
                    print(f"message: {msg.decode('utf-8')}")
                    # データを返信
                    client_socket.send('Success to receive message'.encode('utf-8'))
                except KeyboardInterrupt:
                    self.socket.close()
                    exit()
```
次にクライアント側の処理を書いていきます。  
サーバーと同様に、送信するプロトコルが`TCP`なのか`UDP`なのかで処理を分けます。
```python:client-server.py
# クライアントの処理
    def send(self):
        # UDPの処理
        if self.args.udp:
            # ソケットを作成（IP4, UDP）
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
            while True:
                try:
                    print('Input any messages, or [Ctrl + c] to exit')
                    message = input()
                    # データを送信
                    self.socket.sendto(message.encode('utf-8'), (self.args.target,self.args.port))
                    # データを受信
                    msg, client_address = self.socket.recvfrom(1024)
                    print(f"message: {msg.decode('utf-8')}  from: {client_address}")
                except KeyboardInterrupt:
                    self.socket.close()
                    exit()
        # TCPの処理
        else:
            # ソケットを作成（IP4, TCP）
            self.socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
            # 接続
            self.socket.connect((self.args.target, self.args.port))
            print(f"Connection with {self.args.target, self.args.port} !")
            while True:
                try:
                    print('Input any messages, or [Ctrl + c] to exit')
                    message = input()
                    # データを送信
                    self.socket.send(message.encode())
                    # データを受信
                    msg = self.socket.recv(1024)
                    print(f"message: {msg.decode('utf-8')} ")
                except KeyboardInterrupt:
                    self.socket.close()
                    exit()
```
これでクライアントサーバー機能は完成です。
実行するにはターミナルを２つ立ち上げてコマンドを実行します。
#### TCP
ターミナル１（サーバー）　  ：`python client-server.py -l -t 127.0.0.1 -p 8000`    
ターミナル２（クライアント）：`python client-server.py -t 127.0.0.1 -p 8000`  

上記のように実行すると、TCPプロトコルを使ってターミナル２で入力した文字列がターミナル１に送信できるようになります。  

#### UDP
ターミナル１（サーバー）　  ：`python client-server.py -l -t 127.0.0.1 -p 8000 -u`    
ターミナル２（クライアント）：`python client-server.py -t 127.0.0.1 -p 8000 -u`  
  

### 3.2 ポートスキャン機能
### 3.3 ファイル転送機能
### 3.4 コマンド実行機能(バックドア)
## 4. 参考