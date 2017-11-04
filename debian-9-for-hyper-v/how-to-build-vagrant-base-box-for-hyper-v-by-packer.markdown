Packer を使って Hyper-V 向けの Vagrant のベースボックス (Debian) を作成する方法
==========

## 前提知識

* Packer
* Hyper-V
* Vagrant
* Debian
    * Preseed

## Packer で Vagrant から使うための Hyper-V 用の base box を作成する

* https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/ などでイメージの URL と MD5 チェックサムを取得し、base-box.packer.json に記述。

## Hyper-V と Packer

基本的には

```
==> hyperv-iso: Waiting for SSH to become available...
```

と表示される件は、ホスト OS 側に SSH クライアントがないから？

* 参考 : http://niku.name/Packer%E3%82%92%E4%BD%BF%E3%81%A3%E3%81%A6Vagrant%E3%81%AEBox%E3%82%92%E4%BD%9C%E3%82%8B%E6%96%B9%E6%B3%95%E3%82%92%E4%B8%80%E3%81%A4%E3%81%9A%E3%81%A4%E8%AA%AC%E6%98%8E%E3%81%99%E3%82%8B/index.html

問題発生時にどう調査すればいいか？

* Packer のデバッグ : https://www.packer.io/docs/other/debugging.html
    * `$env:PACKER_LOG=1` でログ出力されるように
* Hyper-V のマシンの画面を表示してそっちで操作

ログを見たら IP アドレスを取得できていなかった。

```
2017/02/18 19:48:48 packer.exe: 2017/02/18 19:48:48 [DEBUG] Error getting SSH address: No ip address.
```

コマンドラインで VM の仮想ネットワークアダプタの IP アドレスを取得しようとして失敗してるっぽい。

https://github.com/mitchellh/packer/blob/45d4cf8b36b8eb5f57e6eea06f14946d3aa2c56e/common/powershell/hyperv/hyperv.go#L750

仮想ネットワークアダプタの IP アドレスを取得するには、ゲスト OS 側で LIS (Linux Integration Services) が有効である必要があるっぽい。

Debian 8.7 Jessie なら、hyperv-daemons パッケージをインストールしさえすれば良さそう。 preseed.cfg なら下記の感じ？

```
d-i pkgsel/include string openssh-server build-essential hyperv-daemons
```

SSH 接続できた後にシャットダウンできない問題。 sudo のパスワード入力がされなくて死ぬ。

```
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 [INFO] Attempting SSH connection...
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 reconnecting to TCP connection for SSH
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 handshaking with SSH
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 handshake complete!
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 [INFO] no local agent socket, will not connect agent
==> hyperv-iso: Connected to SSH!
2017/02/18 20:06:39 ui: ==> hyperv-iso: Connected to SSH!
==> hyperv-iso: Gracefully halting virtual machine...
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 Running the provision hook
2017/02/18 20:06:39 ui: ==> hyperv-iso: Gracefully halting virtual machine...
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 Executing shutdown command: echo 'packer' | sudo -S shutdown -P now
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 opening new ssh session
2017/02/18 20:06:39 packer.exe: 2017/02/18 20:06:39 starting remote command: echo 'packer' | sudo -S shutdown -P now
2017/02/18 20:06:42 packer.exe: 2017/02/18 20:06:42 Remote command exited with '1': echo 'packer' | sudo -S shutdown -P
now
2017/02/18 20:06:42 packer.exe: 2017/02/18 20:06:42 Shutdown stdout:
2017/02/18 20:06:42 packer.exe: 2017/02/18 20:06:42 Shutdown stderr: [sudo] password for vagrant: Sorry, try again.
2017/02/18 20:06:42 packer.exe: [sudo] password for vagrant:
2017/02/18 20:06:42 packer.exe: sudo: 1 incorrect password attempt
2017/02/18 20:06:42 packer.exe: 2017/02/18 20:06:42 Waiting max 5m0s for shutdown to complete
2017/02/18 20:11:42 ui error: ==> hyperv-iso: Timeout while waiting for machine to shut down.
==> hyperv-iso: Timeout while waiting for machine to shut down.
2017/02/18 20:11:45 ui: ==> hyperv-iso: Clean up os dvd drive...
==> hyperv-iso: Clean up os dvd drive...
2017/02/18 20:11:46 ui: ==> hyperv-iso: Unregistering and deleting virtual machine...
==> hyperv-iso: Unregistering and deleting virtual machine...
2017/02/18 20:11:47 ui: ==> hyperv-iso: Deleting output directory...
==> hyperv-iso: Deleting output directory...
2017/02/18 20:11:47 ui: ==> hyperv-iso: Deleting temporary directory...
==> hyperv-iso: Deleting temporary directory...
2017/02/18 20:11:47 ui error: Build 'hyperv-iso' errored: Timeout while waiting for machine to shut down.
Build 'hyperv-iso' errored: Timeout while waiting for machine to shut down.
2017/02/18 20:11:47 Builds completed. Waiting on interrupt barrier...
2017/02/18 20:11:47 machine readable: error-count []string{"1"}
2017/02/18 20:11:47 ui error:
==> Some builds didn't complete successfully and had errors:
2017/02/18 20:11:47 machine readable: hyperv-iso,error []string{"Timeout while waiting for machine to shut down."}
2017/02/18 20:11:47 ui error: --> hyperv-iso: Timeout while waiting for machine to shut down.
2017/02/18 20:11:47 ui:
==> Builds finished but no artifacts were created.
```

* 参考 : https://oitibs.com/install-hyper-v-lis-on-debian-8/

コマンドを `"shutdown_command": "echo 'vagrant' | sudo -S shutdown -P now"` って感じにしたらいけた。 最初がパスワード。

## preseed.cfg の作成方法

Debian インストール後に `debconf-get-selections`
コマンドを使うことで、インストール時に選択した値を保持した preseed.cfg ファイルを取得できるっぽい。

* 参考 : http://askubuntu.com/questions/595826/how-to-create-ubuntu-installation-preseed-file
