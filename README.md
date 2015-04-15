# [stenographer](https://github.com/google/stenographer) についての調査


## stenographer ってなに？

> Stenographer is a packet capture solution which aims to quickly spool all packets to disk, then provide simple, fast access to subsets of those packets

すべてのパケットをキャプチャしてディスクに保存し、かつ、素早く各パケットの情報にアクセスできるようにするソフトウェア

## 構成

TODO: 画像を用意

2つのプロセスで構成されている

### stenographer

- golang 製
- API を提供し、問い合わせに対して、対象のパケット情報を pcap 形式で吐き出す。
- 起動時に `stenotype` を起動する。

### stenotype

- c++ 製
- パケットをキャプチャしてディスクに書き出す
- `cap_net_admin,cap_net_raw,cap_ipc_lock` あたりのキャプチャするのに最低限必要な capability を与えて root でなく一般ユーザーで動かす
- デフォルトでは 1分 1ファイル。
- 複数ディスクへの書き出しや、ディスクの残空き容量でのキャプチャ制御などが出来るようになっている

## クライアント

`stenographer` へ問い合わせをするには、`stenoread`(curl を使った shell script) か curl で直接 API を叩く

例

```console
root@Ubuntu-14:~# stenoread "net 192.168.33.0/24" -n icmp
Running stenographer query 'net 192.168.33.0/24', piping to 'tcpdump -n icmp'
reading from file /dev/stdin, link-type EN10MB (Ethernet)
20:03:39.489053 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 1, length 64
20:03:39.489176 IP 192.168.33.1 > 192.168.33.8: ICMP echo reply, id 2578, seq 1, length 64
20:03:40.487614 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 2, length 64
20:03:40.487978 IP 192.168.33.1 > 192.168.33.8: ICMP echo reply, id 2578, seq 2, length 64
20:03:41.486630 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 3, length 64
20:03:41.487092 IP 192.168.33.1 > 192.168.33.8: ICMP echo reply, id 2578, seq 3, length 64
20:03:42.488469 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 4, length 64
20:03:42.488983 IP 192.168.33.1 > 192.168.33.8: ICMP echo reply, id 2578, seq 4, length 64
20:03:43.489915 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 5, length 64
20:03:43.490329 IP 192.168.33.1 > 192.168.33.8: ICMP echo reply, id 2578, seq 5, length 64
20:03:44.490250 IP 192.168.33.8 > 192.168.33.1: ICMP echo request, id 2578, seq 6, length 64
```

## きになるところ

- 直近1分のデータはファイルに書きだされていないので問い合わせできない

