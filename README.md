# Session

Session 套件是取代 Token 和 JWT 的一種方法。這會將客戶端的階段資訊（如登入、購物車商品）以鍵值組的方式暫時存放於伺服端中，且這些資料客戶端無法得知。在負載平衡上此方法則需要與資料庫連結，否則階段會分散保存導致資料不完整而無法使用。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/session
```

# 使用方式

先將 `Sessioner` 傳入 `Use` 來套用 Session 中介軟體後方能在路由中透過 `Set` 和 `Get`（或 `GetString`、`GetInt`⋯等）方式設置、取得客戶端的階段資訊。

```go
package main

import (
	"github.com/go-mego/mego"
	"github.com/go-mego/session"
)

func main() {
	m := mego.New()
	// 套用 Session 中介軟體後就能在路由中使用。
	m.Use(session.Sessioner())
	m.Get("/", func(s *session.Store) string {
		// 透過 `Set` 將此客戶端的 `Level` 設為 `Admin` 供之後的函式存取。
		s.Set("Level", "Admin")
		// 透過 `GetString` 以字串的方式取得 `Level` 的值。
		return "此使用者的權限為：" + s.GetString("Level") // 結果：「此使用者的權限為：Admin」。
	})
	m.Run()
}
```