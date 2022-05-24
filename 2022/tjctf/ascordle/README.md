# ascordle

wordleが解ければflagがレスポンスに含まれる

```js
if (checkWord(word)) {
    return res.json({
        check: true,
        flag: flag,
        sql: false,
        colors: right,
    })
}
```

```js
const checkWord = (word) => {
    const query = db.prepare(`SELECT * FROM answer WHERE word = '${word}'`)
    return typeof query.get() !== 'undefined'
}
```

SQLの検索結果が一つでも返ってくればcheckWordがtrueを返すためSQLiで一つでも返すようにさせようと考えたが、内部のwaf関数でOR or -- = > <が弾かれる

そのため、unionを用いてanswerテーブルを結合させた

`hoge' union select * from answer '`
シングルクオートを合わせておしまい
