# Session [![GoDoc](https://godoc.org/github.com/go-mego/session?status.svg)](https://godoc.org/github.com/go-mego/session)

Session 套件是取代 Token 和 JWT 的一種方法。這會將客戶端的階段資訊（如登入、購物車商品）以鍵值組的方式暫時存放於伺服端中，且這些資料客戶端無法得知。

注意的是這些資料是以客戶端作為獨立區隔的（因此相同鍵名的資料不會在不同瀏覽器、客戶端中相衝），並以 Cookie 中的特殊獨立編號作為識別方式，因此此套件必須與 Cookie 套件一同使用。

在負載平衡上此方法則需要與資料庫連結，否則階段會分散保存導致資料不完整而無法使用。

# 索引

* [安裝方式](#安裝方式)
* [使用方式](#使用方式)
	* [設置資料](#設置資料)
	* [取得資料](#取得資料)
	* [取得所有階段](#取得所有階段)
	* [銷毀階段](#銷毀階段)

# 安裝方式

打開終端機並且透過 `go get` 安裝此套件即可。

```bash
$ go get github.com/go-mego/session
```

# 使用方式

將 `session.New` 和 `cookie.New` 傳入 Mego 引擎中的 `Use` 表示欲使用 Session 和 Cookie 中介軟體作為全域中介軟體，這讓你能在所有路由中使用 Session。

```go
package main

import (
	"github.com/go-mego/cookie"
	"github.com/go-mego/mego"
	"github.com/go-mego/session"
)

func main() {
	m := mego.New()
	// 使用 Cookie 中介軟體，因為 Session 需要。
	m.Use(cookie.New())
	// 將 Session 中介軟體作為全域中介軟體後就能在所有路由中使用。
	m.Use(session.New())
	m.Run()
}
```

Session 也能夠僅用於單一路由中，這讓你能夠在不同路由中有著不同的設置。但這種用法同時也需要安插 Cookie 中介軟體。

```go
func main() {
	m := mego.New()
	// Session 中介軟體也能夠僅用於單路由中，
	// 但同時必須一同使用 Cookie 中介軟體。
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// ...
	})
	m.Run()
}
```

## 設置資料

以 `Set` 替某客戶端設置資料。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `Set` 將此客戶端階段內的 `Level` 資料設為 `Admin`。
		s.Set("Level", "Admin")
	})
	m.Run()
}
```

## 取得資料

透過 `Get` 能夠以 `interface{}` 的方式取得某客戶端的指定階段資料，。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `Get` 以取得 `Level` 的值。
		fmt.Println(s.Get("Level").(string)) // 結果：Admin
	})
	m.Run()
}
```

如果已經預期到指定資料會是什麼型態，可以透過 `GetString`、`GetInt` 等輔助函式取得階段資料。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `GetString` 以字串的方式取得 `Level` 的值。
		// s.GetInt(), s.GetFloat64() ...
		fmt.Println(s.GetString("Level")) // 結果：Admin
	})
	m.Run()
}
```

## 取得所有階段

透過 `GetAll` 以 `map[string]interface` 的資料型態取得某客戶端的所有階段資料。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `GetAll` 以 `map[string]interface` 的形式
		// 取得此客戶端的所有階段資料。
		s.GetAll()
	})
	m.Run()
}
```

## 銷毀資料

如果有個資料是不再被用到的，透過 `Destory` 就可以銷毀該資料。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `Destroy` 銷毀、移除某客戶端的指定階段資料。
		s.Destroy("Level")
	})
	m.Run()
}
```

## 銷毀階段

如果整個階段資料對某客戶端來說再也不需要了，那麼就可以使用 `DestoryAll` 來全數移除某客戶端的階段資料。

```go
func main() {
	m := mego.New()
	m.GET("/", cookie.New(), session.New(), func(s *session.Store) {
		// 透過 `DestroyAll` 銷毀某客戶端的所有階段資料。
		s.DestroyAll()
	})
	m.Run()
}
```