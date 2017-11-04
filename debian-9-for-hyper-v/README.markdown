Hyper-V 向けの Vagrant のベースボックス (Debian 9; 日本向け)
==========

Packer を利用して Hyper-V 向けの Vagrant ベースボックスをビルドするためのプロジェクト。
ゲスト OS は Debian である。

## 準備

* Hyper-V が有効になっていること。
* Packer がインストールされていること。

## 使い方

### ビルドの仕方

PowerShell で、このディレクトリを開いて `packer.exe build .\base-box.packer.json` コマンドを実行する。

進行状況を見るために、Hyper-V マネージャを起動し、上記コマンド実行後に作られる VM に接続すると良い。

ビルドが完了すると、ベースボックスが packer\_hyperv-iso\_hyperv.box に出力される。

### ビルド結果を Vagrant で利用する

`vagrant box add --name debian-9.2.1 .\packer_hyperv-iso_hyperv.box` コマンドを実行する。 (名前は何でもよい。)

あとは Vagrantfile で `config.vm.box = "debian-9.2.1"` という感じで使用できる。

## トラブルシューティング

うまくいかないときは、下記のことを試すと良い。

* Hyper-V マネージャを開き、Packer の対象となっている VM に接続すると、VM がどういう状態にあるのかを見ることができる。

### よくある問題

* 「Download debconf preconfiguration file」 で 「Failed to retrieve the preconfiguration file」 「The file needed for preconfiguration could not be retrieved from http://xxx.xxx.xxx.xxx:xxxx/preseed.cfg. The installation will proceed in non-automated mode.」 と言われる。
    * ホストマシンの方で指定の URL を見ることができるか確認。 → 見ることができないならそもそもの Packer 側の問題か、設定ミス。
    * ホストマシンから見ることができるなら、VM の方の GUI を操作してコンソールを起動して、指定の URL にアクセスできるか確認。
    * できない場合、ネットワーク周りを調査。 例えば `ip a` で IP アドレスを確認するとか。 (何故かホストマシンと同じ IP アドレスになっていることがある。)
        * DHCP がそういう挙動になってることがあるっぽい。
        * 対策？ → https://superuser.com/questions/1137818/hyper-v-virtual-switch-issue-same-ip-on-guest-and-host
        * ホスト側を IP アドレス固定にして、ゲスト側を DHCP での IP アドレス割り当てに任せるととりあえずは回避できるはず。
* 「パッケージマネージャの設定」 で、「別の CD や DVD を検査したければ、それを今挿入してください。」 「別の CD や DVD を検査しますか？」 で止まってしまう
    * preseed.cfg の設定で回避できそう → https://serverfault.com/questions/602218/how-can-i-answer-the-scan-another-cd-prompt-during-debian-install-with-preseed
* ビルドしたベースボックスを Vagrant に追加して新たにボックスを起動すると 「Script: import\_vm\_vmcx.ps1」 「"Missing Checkpoint present."」 というようなエラーメッセージが表示される。
    * https://github.com/hashicorp/packer/issues/5371 これっぽい。 Generation 1 については Packer 1.1.1 で修正済み。
    * Generation 2 については https://github.com/hashicorp/packer/pull/5517 なので、1.1.1 より後のリリース。

## コンソール出力の例

### ビルド時

```
> packer.exe build .\base-box.packer.json
hyperv-iso output will be in this color.

==> hyperv-iso: Creating temporary directory...
==> hyperv-iso: Downloading or copying ISO
    hyperv-iso: Downloading or copying: https://cdimage.debian.org/debian-cd/current/amd64/iso-cd/debian-9.2.1-amd64-netinst.iso
==> hyperv-iso: Starting HTTP server on port 8276
==> hyperv-iso: Creating switch 'Wi-Fi Virtual Switch' if required...
==> hyperv-iso:     switch 'Wi-Fi Virtual Switch' already exists. Will not delete on cleanup...
==> hyperv-iso: Creating virtual machine...
==> hyperv-iso: Enabling Integration Service...
==> hyperv-iso: Setting boot drive to os dvd drive C:\Users\...\base-box-creation\packer_cache\3ce15b337ec2aa70beae71d30311d2fc5ada89ac7eeec7fbf4ed81ecc5eeea5c.iso ...
==> hyperv-iso: Mounting os dvd drive C:\Users\...\base-box-creation\packer_cache\3ce15b337ec2aa70beae71d30311d2fc5ada89ac7eeec7fbf4ed81ecc5eeea5c.iso ...
==> hyperv-iso: Skipping mounting Integration Services Setup Disk...
==> hyperv-iso: Mounting secondary DVD images...
==> hyperv-iso: Configuring vlan...
==> hyperv-iso: Starting the virtual machine...
==> hyperv-iso: Waiting 10s for boot...
==> hyperv-iso: Host IP for the HyperV machine: 192.168.1.5
==> hyperv-iso: Typing the boot command...
==> hyperv-iso: Waiting for SSH to become available...
==> hyperv-iso: Connected to SSH!
==> hyperv-iso: Gracefully halting virtual machine...
==> hyperv-iso: Waiting for vm to be powered down...
==> hyperv-iso: Unmount/delete secondary dvd drives...
==> hyperv-iso: Unmount/delete Integration Services dvd drive...
==> hyperv-iso: Unmount/delete os dvd drive...
==> hyperv-iso: Delete os dvd drives controller 0 location 1 ...
==> hyperv-iso: Exporting vm...
==> hyperv-iso: Compacting disks...
==> hyperv-iso: Copying to output dir...
==> hyperv-iso: Unregistering and deleting virtual machine...
==> hyperv-iso: Deleting temporary directory...
==> hyperv-iso: Running post-processor: vagrant
==> hyperv-iso (vagrant): Creating Vagrant box for 'hyperv' provider
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Hard Disks\debian-9.2.1-amd64.vhdx
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Hard Disks\debian-9.2.1-amd64.vhdx to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Hard Disks\debian-9.2.1-amd64.vhdx
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Hard Disks\debian-9.2.1-amd64_352FA899-5218-4317-BF6D-1A259143E9D3.avhdx
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Hard Disks\debian-9.2.1-amd64_352FA899-5218-4317-BF6D-1A259143E9D3.avhdx to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Hard Disks\debian-9.2.1-amd64_352FA899-5218-4317-BF6D-1A259143E9D3.avhdx
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.VMRS
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.VMRS to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.VMRS
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmcx
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmcx to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmcx
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmgs
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmgs to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmgs
    hyperv-iso (vagrant): Copying: output-hyperv-debian-9.2.1-amd64\Virtual Machines\box.xml
    hyperv-iso (vagrant): Copyed output-hyperv-debian-9.2.1-amd64\Virtual Machines\box.xml to C:\Users\...\AppData\Local\Temp\packer939780895\Virtual Machines\box.xml
    hyperv-iso (vagrant): Compressing: Vagrantfile
    hyperv-iso (vagrant): Compressing: Virtual Hard Disks\debian-9.2.1-amd64.vhdx
    hyperv-iso (vagrant): Compressing: Virtual Hard Disks\debian-9.2.1-amd64_352FA899-5218-4317-BF6D-1A259143E9D3.avhdx
    hyperv-iso (vagrant): Compressing: Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.VMRS
    hyperv-iso (vagrant): Compressing: Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmcx
    hyperv-iso (vagrant): Compressing: Virtual Machines\7F57337F-3D7C-403A-8ACF-19B4D00C68C6.vmgs
    hyperv-iso (vagrant): Compressing: Virtual Machines\box.xml
    hyperv-iso (vagrant): Compressing: metadata.json
Build 'hyperv-iso' finished.

==> Builds finished. The artifacts of successful builds are:
--> hyperv-iso: VM files in directory: ./output-hyperv-debian-9.2.1-amd64
--> hyperv-iso: 'hyperv' provider box: packer_hyperv-iso_hyperv.box
```

### Vagrant へのボックス追加

```
> vagrant box add --name debian-9.2.1 .\packer_hyperv-iso_hyperv.box
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'debian-9.2.1' (v0) for provider:
    box: Unpacking necessary files from: file://C:/Users/.../packer_hyperv-iso_hyperv.box
    box: Progress: 100% (Rate: 722M/s, Estimated time remaining: 0:00:01)
==> box: Successfully added box 'debian-9.2.1' (v0) for 'hyperv'!
```
