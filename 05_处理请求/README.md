# http.Request类型 请求类型
```
// A Request represents an HTTP request received by a server
// or to be sent by a client.
//
// The field semantics differ slightly between client and server
// usage. In addition to the notes on the fields below, see the
// documentation for Request.Write and RoundTripper.
type Request struct {
    // Method specifies the HTTP method (GET, POST, PUT, etc.).
    // For client requests, an empty string means GET.
    //
    // Go's HTTP client does not support sending a request with
    // the CONNECT method. See the documentation on Transport for
    // details.
    Method string
    // URL specifies either the URI being requested (for server
    // requests) or the URL to access (for client requests).
    //
    // For server requests, the URL is parsed from the URI
    // supplied on the Request-Line as stored in RequestURI.  For
    // most requests, fields other than Path and RawQuery will be
    // empty. (See RFC 7230, Section 5.3)
    //
    // For client requests, the URL's Host specifies the server to
    // connect to, while the Request's Host field optionally
    // specifies the Host header value to send in the HTTP
    // request.
    URL *url.URL
    // The protocol version for incoming server requests.
    //
    // For client requests, these fields are ignored. The HTTP
    // client code always uses either HTTP/1.1 or HTTP/2.
    // See the docs on Transport for details.
    Proto      string // "HTTP/1.0"
    ProtoMajor int    // 1
    ProtoMinor int    // 0
    // Header contains the request header fields either received
    // by the server or to be sent by the client.
    //
    // If a server received a request with header lines,
    //
    //  Host: example.com
    //  accept-encoding: gzip, deflate
    //  Accept-Language: en-us
    //  fOO: Bar
    //  foo: two
    //
    // then
    //
    //  Header = map[string][]string{
    //      "Accept-Encoding": {"gzip, deflate"},
    //      "Accept-Language": {"en-us"},
    //      "Foo": {"Bar", "two"},
    //  }
    //
    // For incoming requests, the Host header is promoted to the
    // Request.Host field and removed from the Header map.
    //
    // HTTP defines that header names are case-insensitive. The
    // request parser implements this by using CanonicalHeaderKey,
    // making the first character and any characters following a
    // hyphen uppercase and the rest lowercase.
    //
    // For client requests, certain headers such as Content-Length
    // and Connection are automatically written when needed and
    // values in Header may be ignored. See the documentation
    // for the Request.Write method.
    Header Header
    // Body is the request's body.
    //
    // For client requests, a nil body means the request has no
    // body, such as a GET request. The HTTP Client's Transport
    // is responsible for calling the Close method.
    //
    // For server requests, the Request Body is always non-nil
    // but will return EOF immediately when no body is present.
    // The Server will close the request body. The ServeHTTP
    // Handler does not need to.
    Body io.ReadCloser
    // GetBody defines an optional func to return a new copy of
    // Body. It is used for client requests when a redirect requires
    // reading the body more than once. Use of GetBody still
    // requires setting Body.
    //
    // For server requests, it is unused.
    GetBody func() (io.ReadCloser, error)
    // ContentLength records the length of the associated content.
    // The value -1 indicates that the length is unknown.
    // Values >= 0 indicate that the given number of bytes may
    // be read from Body.
    //
    // For client requests, a value of 0 with a non-nil Body is
    // also treated as unknown.
    ContentLength int64
    // TransferEncoding lists the transfer encodings from outermost to
    // innermost. An empty list denotes the "identity" encoding.
    // TransferEncoding can usually be ignored; chunked encoding is
    // automatically added and removed as necessary when sending and
    // receiving requests.
    TransferEncoding []string
    // Close indicates whether to close the connection after
    // replying to this request (for servers) or after sending this
    // request and reading its response (for clients).
    //
    // For server requests, the HTTP server handles this automatically
    // and this field is not needed by Handlers.
    //
    // For client requests, setting this field prevents re-use of
    // TCP connections between requests to the same hosts, as if
    // Transport.DisableKeepAlives were set.
    Close bool
    // For server requests, Host specifies the host on which the URL
    // is sought. Per RFC 7230, section 5.4, this is either the value
    // of the "Host" header or the host name given in the URL itself.
    // It may be of the form "host:port". For international domain
    // names, Host may be in Punycode or Unicode form. Use
    // golang.org/x/net/idna to convert it to either format if
    // needed.
    // To prevent DNS rebinding attacks, server Handlers should
    // validate that the Host header has a value for which the
    // Handler considers itself authoritative. The included
    // ServeMux supports patterns registered to particular host
    // names and thus protects its registered Handlers.
    //
    // For client requests, Host optionally overrides the Host
    // header to send. If empty, the Request.Write method uses
    // the value of URL.Host. Host may contain an international
    // domain name.
    Host string
    // Form contains the parsed form data, including both the URL
    // field's query parameters and the PATCH, POST, or PUT form data.
    // This field is only available after ParseForm is called.
    // The HTTP client ignores Form and uses Body instead.
    Form url.Values
    // PostForm contains the parsed form data from PATCH, POST
    // or PUT body parameters.
    //
    // This field is only available after ParseForm is called.
    // The HTTP client ignores PostForm and uses Body instead.
    PostForm url.Values
    // MultipartForm is the parsed multipart form, including file uploads.
    // This field is only available after ParseMultipartForm is called.
    // The HTTP client ignores MultipartForm and uses Body instead.
    MultipartForm *multipart.Form
    // Trailer specifies additional headers that are sent after the request
    // body.
    //
    // For server requests, the Trailer map initially contains only the
    // trailer keys, with nil values. (The client declares which trailers it
    // will later send.)  While the handler is reading from Body, it must
    // not reference Trailer. After reading from Body returns EOF, Trailer
    // can be read again and will contain non-nil values, if they were sent
    // by the client.
    //
    // For client requests, Trailer must be initialized to a map containing
    // the trailer keys to later send. The values may be nil or their final
    // values. The ContentLength must be 0 or -1, to send a chunked request.
    // After the HTTP request is sent the map values can be updated while
    // the request body is read. Once the body returns EOF, the caller must
    // not mutate Trailer.
    //
    // Few HTTP clients, servers, or proxies support HTTP trailers.
    Trailer Header
    // RemoteAddr allows HTTP servers and other software to record
    // the network address that sent the request, usually for
    // logging. This field is not filled in by ReadRequest and
    // has no defined format. The HTTP server in this package
    // sets RemoteAddr to an "IP:port" address before invoking a
    // handler.
    // This field is ignored by the HTTP client.
    RemoteAddr string
    // RequestURI is the unmodified request-target of the
    // Request-Line (RFC 7230, Section 3.1.1) as sent by the client
    // to a server. Usually the URL field should be used instead.
    // It is an error to set this field in an HTTP client request.
    RequestURI string
    // TLS allows HTTP servers and other software to record
    // information about the TLS connection on which the request
    // was received. This field is not filled in by ReadRequest.
    // The HTTP server in this package sets the field for
    // TLS-enabled connections before invoking a handler;
    // otherwise it leaves the field nil.
    // This field is ignored by the HTTP client.
    TLS *tls.ConnectionState
    // Cancel is an optional channel whose closure indicates that the client
    // request should be regarded as canceled. Not all implementations of
    // RoundTripper may support Cancel.
    //
    // For server requests, this field is not applicable.
    //
    // Deprecated: Set the Request's context with NewRequestWithContext
    // instead. If a Request's Cancel field and context are both
    // set, it is undefined whether Cancel is respected.
    Cancel <-chan struct{}
    // Response is the redirect response which caused this request
    // to be created. This field is only populated during client
    // redirects.
    Response *Response
    // ctx is either the client or server context. It should only
    // be modified via copying the whole Request using WithContext.
    // It is unexported to prevent people from using Context wrong
    // and mutating the contexts held by callers of the same request.
    ctx context.Context
}
```

# net.URL类型 请求URL
```
scheme://[userinfo@]host/path[?query][#fragment]
scheme:opaque[?query][#fragment]
```

```
// A URL represents a parsed URL (technically, a URI reference).
//
// The general form represented is:
//
//  [scheme:][//[userinfo@]host][/]path[?query][#fragment]
//
// URLs that do not start with a slash after the scheme are interpreted as:
//
//  scheme:opaque[?query][#fragment]
//
// Note that the Path field is stored in decoded form: /%47%6f%2f becomes /Go/.
// A consequence is that it is impossible to tell which slashes in the Path were
// slashes in the raw URL and which were %2f. This distinction is rarely important,
// but when it is, the code should use RawPath, an optional field which only gets
// set if the default encoding is different from Path.
//
// URL's String method uses the EscapedPath method to obtain the path. See the
// EscapedPath method for more details.
type URL struct {
    Scheme     string
    Opaque     string    // encoded opaque data
    User       *Userinfo // username and password information
    Host       string    // host or host:port
    Path       string    // path (relative paths may omit leading slash)
    RawPath    string    // encoded path hint (see EscapedPath method)
    ForceQuery bool      // append a query ('?') even if RawQuery is empty
    RawQuery   string    // encoded query values, without '?'
    Fragment   string    // fragment for references, without '#'
}
```

1. Path 字段  
	获取请求的 URL  
	例如：http://localhost:8080/hello?username=admin&password=123456
	i. 通过 r.URL.Path 只能得到 /hello
2. RawQuery 字段  
	获取请求的 URL 后面?后面的查询字符串  
	例如：http://localhost:8080/hello?username=admin&password=123456
	i. 通过 r.URL.RawQuery 得到的是 username=admin&password=123456

# http.Header类型 请求头
```
// A Header represents the key-value pairs in an HTTP header.
//
// The keys should be in canonical form, as returned by
// CanonicalHeaderKey.
type Header map[string][]string

// Add adds the key, value pair to the header.
// It appends to any existing values associated with key.
// The key is case insensitive; it is canonicalized by
// CanonicalHeaderKey.
func (h Header) Add(key, value string) {
    textproto.MIMEHeader(h).Add(key, value)
}
// Set sets the header entries associated with key to the
// single element value. It replaces any existing values
// associated with key. The key is case insensitive; it is
// canonicalized by textproto.CanonicalMIMEHeaderKey.
// To use non-canonical keys, assign to the map directly.
func (h Header) Set(key, value string) {
    textproto.MIMEHeader(h).Set(key, value)
}
// Get gets the first value associated with the given key. If
// there are no values associated with the key, Get returns "".
// It is case insensitive; textproto.CanonicalMIMEHeaderKey is
// used to canonicalize the provided key. To access multiple
// values of a key, or to use non-canonical keys, access the
// map directly.
func (h Header) Get(key string) string {
    return textproto.MIMEHeader(h).Get(key)
}
// Del deletes the values associated with key.
// The key is case insensitive; it is canonicalized by
// CanonicalHeaderKey.
func (h Header) Del(key string) {
    textproto.MIMEHeader(h).Del(key)
}
// Write writes a header in wire format.
func (h Header) Write(w io.Writer) error {
    return h.write(w, nil)
}
```
1. 获取请求头中的所有信息  
	r.Header  
	得到的结果如下：
    ```
	map[User-Agent:[Mozilla/5.0 (Windows NT 10.0; Win64; x64) 
	AppleWebKit/537.36 (KHTML, like Gecko) Chrome/67.0.3396.62 Safari/537.36] 
	Accept:[text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,ima
	ge/apng,*/*;q=0.8] Accept-Encoding:[gzip, deflate, br] Accept-Language:[zhCN,zh;q=0.9,en-US;q=0.8,en;q=0.7] Connection:[keep-alive] Upgrade-InsecureRequests:[1]]
    ```
2. 获取请求头中的某个具体属性的值，如获取 Accept-Encoding 的值  
	方式一：r.Header[“Accept-Encoding”]  
		i. 得到的是一个字符串切片  
		ii. 结果
		[gzip, deflate, br]  
	方式二：r.Header.Get(“Accept-Encoding”)  
		i. 得到的是字符串形式的值，多个值使用逗号分隔  
		ii. 结果  
		gzip, deflate, br

# io.ReadCloser类型 请求主体Body
```
// ReadCloser is the interface that groups the basic Read and Close methods.
type ReadCloser interface {
    Reader
    Closer
}

// Reader is the interface that wraps the basic Read method.
//
// Read reads up to len(p) bytes into p. It returns the number of bytes
// read (0 <= n <= len(p)) and any error encountered. Even if Read
// returns n < len(p), it may use all of p as scratch space during the call.
// If some data is available but not len(p) bytes, Read conventionally
// returns what is available instead of waiting for more.
//
// When Read encounters an error or end-of-file condition after
// successfully reading n > 0 bytes, it returns the number of
// bytes read. It may return the (non-nil) error from the same call
// or return the error (and n == 0) from a subsequent call.
// An instance of this general case is that a Reader returning
// a non-zero number of bytes at the end of the input stream may
// return either err == EOF or err == nil. The next Read should
// return 0, EOF.
//
// Callers should always process the n > 0 bytes returned before
// considering the error err. Doing so correctly handles I/O errors
// that happen after reading some bytes and also both of the
// allowed EOF behaviors.
//
// Implementations of Read are discouraged from returning a
// zero byte count with a nil error, except when len(p) == 0.
// Callers should treat a return of 0 and nil as indicating that
// nothing happened; in particular it does not indicate EOF.
//
// Implementations must not retain p.
type Reader interface {
    Read(p []byte) (n int, err error)
}

// Closer is the interface that wraps the basic Close method.
//
// The behavior of Close after the first call is undefined.
// Specific implementations may document their own behavior.
type Closer interface {
    Close() error
}
```

# 获取请求体中的信息
```
index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <form action="http://localhost:8080/getBody" method="POST">
    user name:<input type="text" name="username" value="hanzong"><br/>
    password:<input type="password" name="password" value="666666"><br/>
    <input type="submit">
    </form>
</body>
</html>
```

```
main.go
package main
import "fmt"
import "net/http"
func handler(w http.ResponseWriter, r *http.Request) {
    length := r.ContentLength
    body := make([]byte, length)
    r.Body.Read(body)
    fmt.Fprintln(w, "请求体中的内容是：", string(body))
}
func main() {
    http.HandleFunc("/getBody", handler)
    http.ListenAndServe(":8080", nil)
}
```

# url.Values类型 请求值 Form PostForm
```
// Values maps a string key to a list of values.
// It is typically used for query parameters and form values.
// Unlike in the http.Header map, the keys in a Values map
// are case-sensitive.
type Values map[string][]string

// Get gets the first value associated with the given key.
// If there are no values associated with the key, Get returns
// the empty string. To access multiple values, use the map
// directly.
func (v Values) Get(key string) string {
    if v == nil {
        return ""
    }
    vs := v[key]
    if len(vs) == 0 {
        return ""
    }
    return vs[0]
}
// Set sets the key to value. It replaces any existing
// values.
func (v Values) Set(key, value string) {
    v[key] = []string{value}
}
// Add adds the value to key. It appends to any existing
// values associated with key.
func (v Values) Add(key, value string) {
    v[key] = append(v[key], value)
}
// Del deletes the values associated with key.
func (v Values) Del(key string) {
    delete(v, key)
}
// Encode encodes the values into ``URL encoded'' form
// ("bar=baz&foo=quux") sorted by key.
func (v Values) Encode() string {
    if v == nil {
        return ""
    }
    var buf strings.Builder
    keys := make([]string, 0, len(v))
    for k := range v {
        keys = append(keys, k)
    }
    sort.Strings(keys)
    for _, k := range keys {
        vs := v[k]
        keyEscaped := QueryEscape(k)
        for _, v := range vs {
            if buf.Len() > 0 {
                buf.WriteByte('&')
            }
            buf.WriteString(keyEscaped)
            buf.WriteByte('=')
            buf.WriteString(QueryEscape(v))
        }
    }
    return buf.String()
}
```

# 获取请求表单中的信息
如果请求URL和请求Form中的字段不一样，可以使用http.Request.PostForm获取请求Form中的值
```
package main
import "fmt"
import "net/http"
func handler(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    fmt.Fprintln(w, "请求参数为：", r.Form)
}
func main() {
    http.HandleFunc("/getBody", handler)
    http.ListenAndServe(":8080", nil)
}

package main
import "fmt"
import "net/http"
func handler(w http.ResponseWriter, r *http.Request) {
    r.ParseForm()
    fmt.Fprintln(w, "请求参数为：", r.PostForm)
}
func main() {
    http.HandleFunc("/getBody", handler)
    http.ListenAndServe(":8080", nil)
}
```

# http.Request.FormValue(key) 获取表单字段值 
# http.Request.PostFormValue(key) 获取表单字段值 

如果Form中的enctype属性值为"multipart/form-data"，那么PostForm字段无法获取表单中的数据，需要使用MutipartForm字段
Form的enctype属性默认值是"application/x-www-form-urlencoded"，实现上传文件时需要设置为"multipart/form-data"
```
index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <form action="http://localhost:8080/getBody" method="POST" enctype="application/x-www-form-urlencoded">
    user name:<input type="text" name="username" value="hanzong"><br/>
    password:<input type="password" name="password" value="666666"><br/>
    <input type="submit">
    </form>
    <form action="http://localhost:8080/upload" method="POST" enctype="multipart/form-data">
        <input type="text" name="username" value="hanzong"><br/>
        文件：<input type="file" name="photo"><br/>
        <input type="submit">
        </form>
</body>
</html>
```

```
main.go
package main
import "fmt"
import "net/http"
func handler(w http.ResponseWriter, r *http.Request) {
    r.ParseMultipartForm(1000)
    fmt.Fprintln(w, "请求参数为：", r.MultipartForm)
}
func main() {
    http.HandleFunc("/upload", handler)
    http.ListenAndServe(":8080", nil)
}
```

# 打开上传的文件
mime.multipart.Form
```
// Form is a parsed multipart form.
// Its File parts are stored either in memory or on disk,
// and are accessible via the *FileHeader's Open method.
// Its Value parts are stored as strings.
// Both are keyed by field name.
type Form struct {
    Value map[string][]string
    File  map[string][]*FileHeader
}
```

mime.multipart.FileHeader

```
// A FileHeader describes a file part of a multipart request.
type FileHeader struct {
    Filename string
    Header   textproto.MIMEHeader
    Size     int64
    content []byte
    tmpfile string
}
// Open opens and returns the FileHeader's associated File.
func (fh *FileHeader) Open() (File, error) {
    if b := fh.content; b != nil {
        r := io.NewSectionReader(bytes.NewReader(b), 0, int64(len(b)))
        return sectionReadCloser{r}, nil
    }
    return os.Open(fh.tmpfile)
}
```

```
main.go
package main
import (
    "fmt"
    "io/ioutil"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    r.ParseMultipartForm(1000)
    // fmt.Fprintln(w, "请求参数为：", r.MultipartForm)
    fileHeader := r.MultipartForm.File["photo"][0]
    file, err := fileHeader.Open()
    if err != nil {
        fmt.Println(err)
        return
    }
    data, err := ioutil.ReadAll(file)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Fprintln(w, string(data))
}
func main() {
    http.HandleFunc("/upload", handler)
    http.ListenAndServe(":8080", nil)
}
```

如果只上传一个文件，可以使用http.Request.PostFile
```
main.go
package main
import (
    "fmt"
    "io/ioutil"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
    file, _, err := r.FormFile("photo")
    if err != nil {
        fmt.Println(err)
        return
    }
    data, err := ioutil.ReadAll(file)
    if err != nil {
        fmt.Println(err)
        return
    }
    fmt.Fprintln(w, string(data))
}
func main() {
    http.HandleFunc("/upload", handler)
    http.ListenAndServe(":8080", nil)
}
```

# 给客户端响应 http.ResponseWriter
```
// A ResponseWriter interface is used by an HTTP handler to
// construct an HTTP response.
//
// A ResponseWriter may not be used after the Handler.ServeHTTP method
// has returned.
type ResponseWriter interface {
    // Header returns the header map that will be sent by
    // WriteHeader. The Header map also is the mechanism with which
    // Handlers can set HTTP trailers.
    //
    // Changing the header map after a call to WriteHeader (or
    // Write) has no effect unless the modified headers are
    // trailers.
    //
    // There are two ways to set Trailers. The preferred way is to
    // predeclare in the headers which trailers you will later
    // send by setting the "Trailer" header to the names of the
    // trailer keys which will come later. In this case, those
    // keys of the Header map are treated as if they were
    // trailers. See the example. The second way, for trailer
    // keys not known to the Handler until after the first Write,
    // is to prefix the Header map keys with the TrailerPrefix
    // constant value. See TrailerPrefix.
    //
    // To suppress automatic response headers (such as "Date"), set
    // their value to nil.
    Header() Header
    // Write writes the data to the connection as part of an HTTP reply.
    //
    // If WriteHeader has not yet been called, Write calls
    // WriteHeader(http.StatusOK) before writing the data. If the Header
    // does not contain a Content-Type line, Write adds a Content-Type set
    // to the result of passing the initial 512 bytes of written data to
    // DetectContentType. Additionally, if the total size of all written
    // data is under a few KB and there are no Flush calls, the
    // Content-Length header is added automatically.
    //
    // Depending on the HTTP protocol version and the client, calling
    // Write or WriteHeader may prevent future reads on the
    // Request.Body. For HTTP/1.x requests, handlers should read any
    // needed request body data before writing the response. Once the
    // headers have been flushed (due to either an explicit Flusher.Flush
    // call or writing enough data to trigger a flush), the request body
    // may be unavailable. For HTTP/2 requests, the Go HTTP server permits
    // handlers to continue to read the request body while concurrently
    // writing the response. However, such behavior may not be supported
    // by all HTTP/2 clients. Handlers should read before writing if
    // possible to maximize compatibility.
    Write([]byte) (int, error)
    // WriteHeader sends an HTTP response header with the provided
    // status code.
    //
    // If WriteHeader is not called explicitly, the first call to Write
    // will trigger an implicit WriteHeader(http.StatusOK).
    // Thus explicit calls to WriteHeader are mainly used to
    // send error codes.
    //
    // The provided code must be a valid HTTP 1xx-5xx status code.
    // Only one header may be written. Go does not currently
    // support sending user-defined 1xx informational headers,
    // with the exception of 100-continue response header that the
    // Server sends automatically when the Request.Body is read.
    WriteHeader(statusCode int)
}
```

1. 给客户端相应一个字符串
	```
	package main
	import (
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    w.Write([]byte("你的请求我已经收到"))
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
	浏览器显示  
	你的请求我已经收到

	响应报文内容
	```
	HTTP/1.1 200 OK
	Date: Tue, 07 Jan 2020 13:08:11 GMT
	Content-Length: 27
	Content-Type: text/plain; charset=utf-8
	```

2. 给客户端响应一个HTML页面
	```
		package main
		import (
		    "net/http"
		)
		func handler(w http.ResponseWriter, r *http.Request) {
		    html := `<html>
		<head>
		    <title>网页title</title>
		    <meta charset="utf-8"/>
		</head>
		<body>
		    <h1>网页body</h1>
		</body>
		</html>`
		    w.Write([]byte(html))
		}
		func main() {
		    http.HandleFunc("/", handler)
		    http.ListenAndServe(":8080", nil)
		}
	```

3. 给客户端响应JSON格式数据
	```
		package main
		import (
		    "encoding/json"
		    "fmt"
		    "net/http"
		)
		type User struct {
		    ID       int
		    UserName string
		    Password string
		}
		func handler(w http.ResponseWriter, r *http.Request) {
		    w.Header().Set("Content-Type", "application/json")
		    user := User{
		        ID:       1,
		        UserName: "admin",
		        Password: "123456",
		    }
		    json, err := json.Marshal(user)
		    if err != nil {
		        fmt.Println(err)
		        return
		    }
		    w.Write(json)
		}
		func main() {
		    http.HandleFunc("/", handler)
		    http.ListenAndServe(":8080", nil)
		}
	```

4. 让客户端重定向
	```
	package main
	import (
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    w.Header().Set("Location", "https://www.baidu.com")
	    w.WriteHeader(302)
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```

	响应报文
	```
	HTTP/1.1 302 Found
	Location: https://www.baidu.com
	Date: Tue, 07 Jan 2020 13:25:59 GMT
	Content-Length: 0
	```
