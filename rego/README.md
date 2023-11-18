# rego入門

## 直代入
```rego
role = "admin"
```

## if
### 変数名 { ルール } という形式で書くとルール内の評価式が成立する場合に true が変数に代入される。成立しない場合は 変数に何も代入されない
```rego
allow {
	input.user == "blue"
}

# ルールが成立しない場合になにか値を入れいたい、という場合は default キーワードを使用。この場合blue出ない場合falseになる
default allow = false

allow {
	input.user == "blue"
}
```

## if-then
### 変数名 = 値 { ルール } という記述によって、ルールが成立した場合に任意の値を代入
```rego
role = "admin" {
	input.user == "blue"
}
role = "admin" {
	input.user == "orange"
}
```

## if-else-then
```rego
role = "admin" {
	input.user == "blue"
} else = "reader" {
	input.team == "developer"
}
```


## ルール内の変数を利用して代入
### 以下ルールは user が alice ではなかった場合に、 denied_msg にメッセージを代入
```rego
denied_msg = msg {
    input.user != "alice"
    msg := sprintf("%s is not allowed", [input.user])
}
```


### 以下は filtering がオブジェクト型（キーと値を持つ辞書型）になり、src が blue のエントリーがあれば dst をキーとして allowed を代入
```rego
filtering[req.dst] = "allowed" {
    req := input.requests[_]
    req.src == "blue"
}
```
```json
{
	"requests": [
        {"src": "orange", "dst": "10.0.0.1"},
        {"src": "blue", "dst": "10.0.0.2"},
        {"src": "red", "dst": "10.0.0.3"}
    ]
}
```
```bash
{
    "filtering": {
        "10.0.0.2": "allowed"
    }
}
```
### 変数名[キー名] { ルール } という記述をすると、ルールが真になるときにキー名の集合が作成
```rego
filtering[req.dst] {
    req := input.requests[_]
    req.src != "blue"
}
```
```bash
{
    "filtering": [
        "10.0.0.1",
        "10.0.0.3"
    ]
}
```


##　例
```rego
deny_msg[msg] {
    input.request.user != "blue"
    msg := "User is not blue"
}

deny_msg[msg] {
    input.request.path != "/api/test"
    msg := sprintf("Requested path %s is not allowed", [input.request.path])
}
```

```bash
{
    "deny_msg": [
        "Requested path /some/where is not allowed",
        "User is not blue"
    ]
}
```
