# 处理静态文件
```
http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("views/static"))))
```

# 常用函数
```
http.Handle、http.StripPrefix、http.FileServer、http.Dir
// Handle registers the handler for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func Handle(pattern string, handler Handler) { DefaultServeMux.Handle(pattern, handler) }
// HandleFunc registers the handler function for the given pattern
// in the DefaultServeMux.
// The documentation for ServeMux explains how patterns are matched.
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
    DefaultServeMux.HandleFunc(pattern, handler)
}
```
```
// StripPrefix returns a handler that serves HTTP requests
// by removing the given prefix from the request URL's Path
// and invoking the handler h. StripPrefix handles a
// request for a path that doesn't begin with prefix by
// replying with an HTTP 404 not found error.
func StripPrefix(prefix string, h Handler) Handler {
    if prefix == "" {
        return h
    }
    return HandlerFunc(func(w ResponseWriter, r *Request) {
        if p := strings.TrimPrefix(r.URL.Path, prefix); len(p) < len(r.URL.Path) {
            r2 := new(Request)
            *r2 = *r
            r2.URL = new(url.URL)
            *r2.URL = *r.URL
            r2.URL.Path = p
            h.ServeHTTP(w, r2)
        } else {
            NotFound(w, r)
        }
    })
}
```
```
// FileServer returns a handler that serves HTTP requests
// with the contents of the file system rooted at root.
//
// To use the operating system's file system implementation,
// use http.Dir:
//
//     http.Handle("/", http.FileServer(http.Dir("/tmp")))
//
// As a special case, the returned file server redirects any request
// ending in "/index.html" to the same path, without the final
// "index.html".
func FileServer(root FileSystem) Handler {
    return &fileHandler{root}
}
```
```
// A Dir implements FileSystem using the native file system restricted to a
// specific directory tree.
//
// While the FileSystem.Open method takes '/'-separated paths, a Dir's string
// value is a filename on the native file system, not a URL, so it is separated
// by filepath.Separator, which isn't necessarily '/'.
//
// Note that Dir will allow access to files and directories starting with a
// period, which could expose sensitive directories like a .git directory or
// sensitive files like .htpasswd. To exclude files with a leading period,
// remove the files/directories from the server or create a custom FileSystem
// implementation.
//
// An empty Dir is treated as ".".
type Dir string
```

```
./views/static/css/style.css
h1{
    color: red;
}
```
```
./views/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <link rel="stylesheet" href="static/css/style.css">
</head>
<body>
    <h1>一段红色的文字</h1>
</body>
</html>
```
```
./main.go
package main
import (
    "html/template"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    t := template.Must(template.ParseFiles("views/index.html"))
    t.Execute(w, nil)
}
func main() {
    http.Handle("/static/", http.StripPrefix("/static/", http.FileServer(http.Dir("views/static"))))
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

