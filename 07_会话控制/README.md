# 会话控制
HTTP是无状态协议，服务器不能区分请求是不是来自于同一个客户端。

# Cookie
Cookie是服务器保存在浏览器上的信息。浏览器每次向服务器请求时都会将Cookie发送给服务器

# Cookie的运行原理
	1. 浏览器第一次向服务器发送请求时在服务器端创建Cookie
	2. 服务器端创建的Cookie以响应头的而方式发送给浏览器
	3. 之后浏览器发送请求时带上Cookie
	4. 服务器端通过Cookie区分不同的用户

# Cookie的用途
	1. 广告推荐
	2. 免登录

# Http.Cookie及常用函数

```
// A Cookie represents an HTTP cookie as sent in the Set-Cookie header of an
// HTTP response or the Cookie header of an HTTP request.
//
// See https://tools.ietf.org/html/rfc6265 for details.
type Cookie struct {
    Name  string
    Value string
    Path       string    // optional
    Domain     string    // optional
    Expires    time.Time // optional
    RawExpires string    // for reading cookies only
    // MaxAge=0 means no 'Max-Age' attribute specified.
    // MaxAge<0 means delete cookie now, equivalently 'Max-Age: 0'
    // MaxAge>0 means Max-Age attribute present and given in seconds
    MaxAge   int
    Secure   bool
    HttpOnly bool
    SameSite SameSite
    Raw      string
    Unparsed []string // Raw text of unparsed attribute-value pairs
}
// SetCookie adds a Set-Cookie header to the provided ResponseWriter's headers.
// The provided cookie must have a valid Name. Invalid cookies may be
// silently dropped.
func SetCookie(w ResponseWriter, cookie *Cookie) {
    if v := cookie.String(); v != "" {
        w.Header().Add("Set-Cookie", v)
    }
}
```

# 创建Cookie
```
./main.go
package main
import "net/http"
func handler(w http.ResponseWriter, r *http.Request) {
    cookie1 := http.Cookie{
        Name:     "user1",
        Value:    "zhangsan",
        HttpOnly: true,
    }
    cookie2 := http.Cookie{
        Name:     "user2",
        Value:    "lisi",
        HttpOnly: true,
    }
    // w.Header().Set("Set-Cookie", cookie1.String())
    // w.Header().Add("Set-Cookie", cookie2.String())
    http.SetCookie(w, &cookie1)
    http.SetCookie(w, &cookie2)
}
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

第一次请求
```
GET http://localhost:8080/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-Hans-CN,zh-Hans;q=0.8,en-US;q=0.5,en;q=0.3
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
Accept-Encoding: gzip, deflate
Host: localhost:8080
Connection: Keep-Alive
```

第一次响应
```
HTTP/1.1 200 OK
Set-Cookie: user1=zhangsan; HttpOnly
Set-Cookie: user2=lisi; HttpOnly
Date: Fri, 10 Jan 2020 05:24:16 GMT
Content-Length: 0
```

第二次请求
```
GET http://localhost:8080/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-Hans-CN,zh-Hans;q=0.8,en-US;q=0.5,en;q=0.3
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
Accept-Encoding: gzip, deflate
Host: localhost:8080
Connection: Keep-Alive
Cookie: user1=zhangsan; user2=lisi
```

# 读取Cookie
```
package main
import (
    "fmt"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    cookie1 := http.Cookie{
        Name:     "user1",
        Value:    "zhangsan",
        HttpOnly: true,
    }
    cookie2 := http.Cookie{
        Name:     "user2",
        Value:    "lisi",
        HttpOnly: true,
    }
    cookies := r.Header["Cookie"]
    if cookies == nil {
        // w.Header().Set("Set-Cookie", cookie1.String())
        // w.Header().Add("Set-Cookie", cookie2.String())
        http.SetCookie(w, &cookie1)
        http.SetCookie(w, &cookie2)
    }
    fmt.Fprintln(w, cookies)
}
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

# 设置Cookie有效时间
```
./main.go
package main
import (
    "fmt"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    cookie1 := http.Cookie{
        Name:     "user1",
        Value:    "zhangsan",
        HttpOnly: true,
        MaxAge:   10,
    }
    cookies := r.Header["Cookie"]
    if cookies == nil {
        http.SetCookie(w, &cookie1)
    }
    fmt.Fprintln(w, cookies)
}
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

第一次请求
```
GET http://localhost:8080/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-Hans-CN,zh-Hans;q=0.8,en-US;q=0.5,en;q=0.3
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
Accept-Encoding: gzip, deflate
Host: localhost:8080
Connection: Keep-Alive
```

第一次响应
```
HTTP/1.1 200 OK
Set-Cookie: user1=zhangsan; Max-Age=10; HttpOnly
Date: Fri, 10 Jan 2020 05:37:43 GMT
Content-Length: 3
Content-Type: text/plain; charset=utf-8

[]
```

第二次请求
```
GET http://localhost:8080/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-Hans-CN,zh-Hans;q=0.8,en-US;q=0.5,en;q=0.3
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
Accept-Encoding: gzip, deflate
Host: localhost:8080
Connection: Keep-Alive
Cookie: user1=zhangsan
```
过期的请求
```
GET http://localhost:8080/ HTTP/1.1
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-Hans-CN,zh-Hans;q=0.8,en-US;q=0.5,en;q=0.3
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.102 Safari/537.36 Edge/18.18362
Accept-Encoding: gzip, deflate
Host: localhost:8080
Connection: Keep-Alive
```
# Session
为了减少浏览器请求中的Cookie的数量，可以使用Session。Session是一个特殊的Cookie，这个Cookie对应服务器端的一个Session，通过这个Cookie可以获取保存其他信息的Session

# Session运行原理
	1. 浏览器第一次向服务器发送请求时，服务器端创建Session，设置一个GUID（可以通过UUID生成）
	2. 服务器端创建一个Cookie，把Cookie的值设为Session的ID值，并将Cookie发送给浏览器
	3. 浏览器之后发送请求带上Cookie
	4. 服务器端获取到Cookie，根据Cookie的值找到服务器中对应的Session
