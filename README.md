# Percona XtraDB Cluster Galera Arbitrator

## 概要

Galera クラスタ用の arbitrator 構築用の ansible role.

DBクラスタのノード数が偶数の場合、local network のリンクだけ切れた場合に、スプリットブレイン状態になる。
デフォルトのDBクラスタの設定だと、スプリットブレインになるとサービスが止まってしまうため、arbitrator を追加して、ノード数を奇数にして、多数決ができるようにしないといけない。

## 注意点

実際に、DB上でデータの読み書き処理はしないので、悪影響はでにくい。
出にくいが、ない訳ではないので、以下の点には配慮すること

* ネットワーク性能は高めに（10G NIC だと遅延が少なく安心）
* CPU性能はそこそこ注意すること

なお、ディスク性能は要求されない。


## 動作確認バージョン
- Ubuntu 20.04 (focal)
- ansible >= 2.8
- Jinja2 2.10.3

## 使い方 (garbd, arbitrator)

### 起動
```
$ sudo systemctl start garbd.service 
```
### 停止
```
$ sudo systemctl stop garbd.service 
```
### log の確認
```
$ journalctl -u garbd
```

## 使い方 (ansible)

### Role variables

```yaml
### インストール設定 ###############################################################################
## 基本設定
pxc_garbd_install_flag: True  # インストールフラグ
pxc_garbd_package_name: "percona-release_latest.generic_all.deb"
pxc_garbd_version: "5.7"
pxc_garbd_config_path: "/etc/default/garbd"
pxc_garbd_pxc_name: "{{ pxc_hosts | default('link-u_pxc') }}"

## クラスタ内の IP アドレスリスト
pxc_garbd_host_ips: []
## 例1: クラスタ内の IP アドレスを直接リストする.
# pxc_garbd_host_ips:
#   - "192.0.2.11"
#   - "192.0.2.12"
#   - "192.0.2.13"
#
## 例2: production グループの local_ipv4 アドレスをリストする
# pxc_garbd_host_ips: "{{ groups['production'] | map('extract', hostvars, 'local_ipv4') | list }}"

pxc_garbd_galera_options:
  base_dir:             "/var/lib/galera"    ## クラスタノードの同期順序を記録するために必要
#  gmcast.listen_addr:   "{{ local_ipv4 }}"  ## listen アドレスの制限 (セキュア化)
```

### Example playbook

```yaml
- hosts:
    - pxc_garbd
  become: yes
  vars:
    pxc_garbd_host_ips: "{{ groups['production'] | map('extract', hostvars, 'local_ipv4') | list }}"
  roles:
    - { role: percona-cluster-garbd, tags: ["percona-cluster-garbd"] }
```

## License
GPLv2

※ GPLで頒布されている [percona-xtradb-cluster](https://github.com/percona/percona-xtradb-cluster/) のファイルを改変した [templates/garbd.j2](./templates/garbd.j2) を含んでいる.
