# 简介
	• net/http包调用ListenAndServe函数，传入网络地址和负责处理请求的处理器，创建一个服务器
	• 网路地址为空字符串，服务器默认使用80端口进行网路连接
	• 处理器为nil，服务器使用默认的多路复用器DefaultServeMux
	• 多路复用器接收到用户请求后根据URL判断使用哪个处理器来处理，找到后会重定向到对应的处理器来处理请求

# 使用默认的多路复用器DefaultServeMux
```
package main
import "net/http"
import "fmt"
type MyHandler struct{}
func (m *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "正在通过处理器函数处理你的请求")
}
func main() {
    http.Handle("/", &MyHandler{})
    fmt.Println("8080端口已开始监听")
    http.ListenAndServe(":8080", nil)
}
```

# 实现ServerHTTP方法就实现了Handler接口
```
type Handler interface {
    ServeHTTP(ResponseWriter, *Request)
}
```

# 使用Server结构对服务器进行配置
```
package main
import (
    "fmt"
    "net/http"
    "time"
)
type MyHandler struct{}
func (m *MyHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "正在通过处理器函数处理你的请求")
}
func main() {
    server := http.Server{
        Addr:        ":8080",
        Handler:     &MyHandler{},
        ReadTimeout: 2 * time.Second,
    }
    fmt.Println("8080端口已开始监听")
    server.ListenAndServe()
}
```

# 使用自己创建的多路复用器
```package main
import (
    "fmt"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintln(w, "正在通过自己创建的多路复用器处理你的请求")
}
func main() {
    mux := http.NewServeMux()
    mux.HandleFunc("/myMux", handler)
    http.ListenAndServe(":8080", mux)
}
```

ServeMux类型是HTTP请求的多路转接器，它会将每一个请求的URL与一个注册模式的列表进行匹配，并调用和URL最匹配的模式的处理器
```
type ServeMux struct {
    mu    sync.RWMutex
    m     map[string]muxEntry
    es    []muxEntry // slice of entries sorted from longest to shortest.
    hosts bool       // whether any patterns contain hostnames
}
```