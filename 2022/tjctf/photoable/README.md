# photoable

imageidにディレクトリトラバーサルしそうな文字列を入れたらflagが入手できそうなことがわかる

```js
app.get("/image/:imageid/download", (req, res) => {
  let { imageid } = req.params;

  res.sendFile(path.join(__dirname, `photobucket/${getFileName(imageid)}`));
});
```


愚直に`/image/../flag.txt/download`とするとアクセスができない

内部でディレクトリトラバーサルされるように/をエンコードし、`/image/..%2fflag.txt/download`とすることでflagを入手
