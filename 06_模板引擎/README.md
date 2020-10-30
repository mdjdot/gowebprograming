# 模板引擎
- go提供了 text/template 和 html/template 两个模板引擎
- 通过模板引擎把数据和模板组合成HTML
- 处理器调用模板引擎并将生成的HTML返回给客户端

# 使用模板
hello.html

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>模板文件</title>
</head>
<body>
    <!-- 嵌入动作 -->
    {{.}}
</body>
</html>
```

main.go
```
package main
import (
    "fmt"
    "html/template"
    "net/http"
)
func handler(w http.ResponseWriter, r *http.Request) {
	// t := template.New("hello.html")
	// t, err := t.ParseFiles("hello.html")
	// t := template.Must(template.ParseFiles("hello.html"))
    t, err := template.ParseFiles("./hello.html")
    if err != nil {
        fmt.Println(err)
        w.WriteHeader(500)
    }
    t.Execute(w, "Hello World!")
}
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

# 常用方法
html.template.Template
```
// Template is a specialized Template from "text/template" that produces a safe
// HTML document fragment.
type Template struct {
    // Sticky error if escaping fails, or escapeOK if succeeded.
    escapeErr error
    // We could embed the text/template field, but it's safer not to because
    // we need to keep our version of the name space and the underlying
    // template's in sync.
    text *template.Template
    // The underlying template's parse tree, updated to be HTML-safe.
    Tree       *parse.Tree
    *nameSpace // common to all associated templates
}
// Execute applies a parsed template to the specified data object,
// writing the output to wr.
// If an error occurs executing the template or writing its output,
// execution stops, but partial results may already have been written to
// the output writer.
// A template may be executed safely in parallel, although if parallel
// executions share a Writer the output may be interleaved.
func (t *Template) Execute(wr io.Writer, data interface{}) error {
    if err := t.escape(); err != nil {
        return err
    }
    return t.text.Execute(wr, data)
}
// Parse parses text as a template body for t.
// Named template definitions ({{define ...}} or {{block ...}} statements) in text
// define additional templates associated with t and are removed from the
// definition of t itself.
//
// Templates can be redefined in successive calls to Parse,
// before the first use of Execute on t or any associated template.
// A template definition with a body containing only white space and comments
// is considered empty and will not replace an existing template's body.
// This allows using Parse to add new named template definitions without
// overwriting the main template body.
func (t *Template) Parse(text string) (*Template, error) {
    if err := t.checkCanParse(); err != nil {
        return nil, err
    }
    ret, err := t.text.Parse(text)
    if err != nil {
        return nil, err
    }
    // In general, all the named templates might have changed underfoot.
    // Regardless, some new ones may have been defined.
    // The template.Template set has been updated; update ours.
    t.nameSpace.mu.Lock()
    defer t.nameSpace.mu.Unlock()
    for _, v := range ret.Templates() {
        name := v.Name()
        tmpl := t.set[name]
        if tmpl == nil {
            tmpl = t.new(name)
        }
        tmpl.text = v
        tmpl.Tree = v.Tree
    }
    return t, nil
}
// New allocates a new HTML template associated with the given one
// and with the same delimiters. The association, which is transitive,
// allows one template to invoke another with a {{template}} action.
//
// If a template with the given name already exists, the new HTML template
// will replace it. The existing template will be reset and disassociated with
// t.
func (t *Template) New(name string) *Template {
    t.nameSpace.mu.Lock()
    defer t.nameSpace.mu.Unlock()
    return t.new(name)
}
// Must is a helper that wraps a call to a function returning (*Template, error)
// and panics if the error is non-nil. It is intended for use in variable initializations
// such as
//  var t = template.Must(template.New("name").Parse("html"))
func Must(t *Template, err error) *Template {
    if err != nil {
        panic(err)
    }
    return t
}
// ParseFiles creates a new Template and parses the template definitions from
// the named files. The returned template's name will have the (base) name and
// (parsed) contents of the first file. There must be at least one file.
// If an error occurs, parsing stops and the returned *Template is nil.
//
// When parsing multiple files with the same name in different directories,
// the last one mentioned will be the one that results.
// For instance, ParseFiles("a/foo", "b/foo") stores "b/foo" as the template
// named "foo", while "a/foo" is unavailable.
func ParseFiles(filenames ...string) (*Template, error) {
    return parseFiles(nil, filenames...)
}
// ParseFiles parses the named files and associates the resulting templates with
// t. If an error occurs, parsing stops and the returned template is nil;
// otherwise it is t. There must be at least one file.
//
// When parsing multiple files with the same name in different directories,
// the last one mentioned will be the one that results.
//
// ParseFiles returns an error if t or any associated template has already been executed.
func (t *Template) ParseFiles(filenames ...string) (*Template, error) {
    return parseFiles(t, filenames...)
}
// ParseGlob creates a new Template and parses the template definitions from
// the files identified by the pattern. The files are matched according to the
// semantics of filepath.Match, and the pattern must match at least one file.
// The returned template will have the (base) name and (parsed) contents of the
// first file matched by the pattern. ParseGlob is equivalent to calling
// ParseFiles with the list of files matched by the pattern.
//
// When parsing multiple files with the same name in different directories,
// the last one mentioned will be the one that results.
func ParseGlob(pattern string) (*Template, error) {
    return parseGlob(nil, pattern)
}
```

# 解析模板
- t, err := template.ParseFiles("hello.html")
等价于
	```
	t := template.New("hello.html")
	t, err :=t.ParseFiles("hello.html")
	```
- t := template.Must(template.ParseFiles("hello.html"))
- template.ParseGlob("*.html")可以创建并解析匹配的文件里的模板，内容为解析后的第一个文件的内容
	
# 执行模板
```
// Template is a specialized Template from "text/template" that produces a safe
// HTML document fragment.
type Template struct {
    // Sticky error if escaping fails, or escapeOK if succeeded.
    escapeErr error
    // We could embed the text/template field, but it's safer not to because
    // we need to keep our version of the name space and the underlying
    // template's in sync.
    text *template.Template
    // The underlying template's parse tree, updated to be HTML-safe.
    Tree       *parse.Tree
    *nameSpace // common to all associated templates
}
// Execute applies a parsed template to the specified data object,
// writing the output to wr.
// If an error occurs executing the template or writing its output,
// execution stops, but partial results may already have been written to
// the output writer.
// A template may be executed safely in parallel, although if parallel
// executions share a Writer the output may be interleaved.
func (t *Template) Execute(wr io.Writer, data interface{}) error {
    if err := t.escape(); err != nil {
        return err
    }
    return t.text.Execute(wr, data)
}
// ExecuteTemplate applies the template associated with t that has the given
// name to the specified data object and writes the output to wr.
// If an error occurs executing the template or writing its output,
// execution stops, but partial results may already have been written to
// the output writer.
// A template may be executed safely in parallel, although if parallel
// executions share a Writer the output may be interleaved.
func (t *Template) ExecuteTemplate(wr io.Writer, name string, data interface{}) error {
    tmpl, err := t.lookupAndEscapeTemplate(name)
    if err != nil {
        return err
    }
    return tmpl.text.Execute(wr, data)
}
```

Execute方法将解析好的模板应用到data上，并输出写入wr  
- template.Execute只会调用第一个模板  
- template.ExecuteTemplate可以调用其他模板  
t := template.ParseFiles("hello.html", "hello2.html")  
T.ExecuteTemplate(w, "hello2.html"，"显示hello2.html")
	
# 动作
模板的动作是嵌入到模板的命令  
{{.}}：代表传递给模板的数据  
-  条件动作  
	arg是传递给条件动作的参数，该值可以是一个字符串常量、一个变量、一个返回单个值的函数获取方法等  
	```
	{{if arg}}
	要显示的内容
	{{end}}
	```
	```
	{{if arg}}
	要显示的内容
	{{else}}
	要显示的内容
	{{end}}
	```
	```
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    <!-- 嵌入动作 -->
	    {{if .}}
	    <h1>传入了数据</h1>
	    {{else}}
	    <h1>未传入数据</h1>
	    {{end}}
	</body>
	</html>
	```
	```
	package main
	import (
	    "fmt"
	    "html/template"
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    t, err := template.ParseFiles("./hello.html")
	    if err != nil {
	        fmt.Println(err)
	        w.WriteHeader(500)
	    }
	    // t.Execute(w, "Hello World!")
	    t.Execute(w, nil)
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
-  迭代动作  
	迭代动作可以对数组、切片、映射或者通道进行迭代  
	迭代数组
	```
	{{range .}}
	遍历到的元素是{{.}}
	{{end}}
	```
	```
	{{range .}}
	遍历到的元素是{{.}}
	{{else}}
	没有任何元素
	{{end}}
	```	
	迭代结构体
	```
	{{range .Name}}
	{{else}}
	没有任何元素
	{{end}}
	```
	迭代Map
	```
	{{range &k,$v:=.}}
	键是{{$k}}，值是{{$v}}
	{{else}}
	没有任何元素
	{{end}}
	```
	迭代管道
	```
	{{c1|c2|c3}}
	```
	```
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    <!-- 嵌入动作 -->
	    {{range .}}
	    <a href="#">{{.Name}}{{.Age}}</a><br/>
	    {{else}}
	    没有遍历到任何内容
	    {{end}}
	</body>
	</html>
	```
	```
	package main
	import (
	    "fmt"
	    "html/template"
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    t, err := template.ParseFiles("./hello.html")
	    if err != nil {
	        fmt.Println(err)
	        w.WriteHeader(500)
	    }
	    stars := []struct {
	        Name string
	        Age  int
	    }{
	        {Name: "马蓉", Age: 18},
	        {Name: "李小璐", Age: 20},
	        {Name: "白百合", Age: 17},
	    }
	    // stars := []string{"马蓉", "李小璐", "白百合"}
	    t.Execute(w, stars)
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
- 设置动作  
	设置动作允许在指定的范围内对data设置值
	```
	{{with arg}}
	为传过来的数据设置的新值是{{.}}
	{{end}}
	```
	```
	{{with arg}}
	为传过来的数据设置的新值是{{.}}
	{{else}}
	传过来的数据仍然是{{.}}
	{{end}}
	```
	```
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    <!-- 嵌入动作 -->
	    <div>得到的数据是：{{.}}</div>
	    {{with "太子"}}
	    <div>替换之后的数据是：{{.}}</div>
	    {{end}}
	    <hr/>
	    {{with ""}}
	    <div>看一下现在的数据是：{{.}}</div>
	    {{else}}
	    <div>数据没有被替换，还是：{{.}}</div>
	    {{end}}
	</body>
	</html>
	```
	```
	package main
	import (
	    "fmt"
	    "html/template"
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    t, err := template.ParseFiles("./hello.html")
	    if err != nil {
	        fmt.Println(err)
	        w.WriteHeader(500)
	    }
	    t.Execute(w, "狸猫")
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
	
- 包含动作  
	包含动作允许用户在一个模板里面包含另一个模板，从而构建出嵌套的模板
	```
	{{template "name"}}
	```
	```
	{{template "name" arg}}
	```

	```
	./hello.html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    <!-- 嵌入动作 -->
	    <div>从后台得到的数据是：{{.}}</div>
	    <!-- 包含hello2.html模板 -->
	    {{template "hello2.html"}}
	    <div>hello.html文件内容结束</div>
	    <hr/>
	    <div>将hello.html模板文件中的数据传递给hello2.html模板文件</div>
	    {{template "hello2.html" .}}
	</body>
	</html>
	```
	```
	./hello2.html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>hello2模板文件</title>
	</head>
	<body>
	    <!-- 嵌入动作 -->
	    <div>hello2.html模板文件中的数据是：{{.}}</div>
	</body>
	</html>
	```
	```
	./main.go
	package main
	import (
	    "fmt"
	    "html/template"
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    t, err := template.ParseFiles("./hello.html", "./hello2.html")
	    if err != nil {
	        fmt.Println(err)
	        w.WriteHeader(500)
	    }
	    t.Execute(w, "测试包含")
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
	• 定义动作
	定义动作达到不同网页使用相同部分，如导航栏、版权信息、联系信息等
```
	{{define "layout"}}
	{{end}}
```
```
	<!-- 定义模板 -->
	{{define "model"}}
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>Document</title>
	</head>
	<body>
	    
	</body>
	</html>
	{{end}}
```

	在一个模板文件中定义多个模板
```
    ./Index.html
	<!-- 定义模板 -->
	{{define "model"}}
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>Document</title>
	</head>
	<body>
	    {{template "content"}}
	</body>
	</html>
	{{end}}
	{{define "content"}}
	<a href="#">点我有惊喜</a>
	{{end}}
```
```	
	./main.go
	package main
	import (
	    "fmt"
	    "html/template"
	    "net/http"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    t, err := template.ParseFiles("./index.html")
	    if err != nil {
	        fmt.Println(err)
	        w.WriteHeader(500)
	    }
	    t.ExecuteTemplate(w, "model", "")
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
```

在不同模板文件中定义同名的模板
```
./hello.html
<!-- 定义模板 -->
{{define "model"}}
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>模板文件</title>
</head>
<body>
    {{template "content"}}
</body>
</html>
{{end}}
```
```
./content1.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>模板文件</title>
</head>
<body>
    {{define "content"}}
        <h1>我是content1.html 模板文件中的内容</h1>
    {{end}}
</body>
</html>
```
```
./content2.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>模板文件</title>
</head>
<body>
    {{define "content"}}
    <h1>我是content2.html 模板文件中的内容</h1>
{{end}}
</body>
</html>
```
```	
./main.go
package main
import (
    "html/template"
    "math/rand"
    "net/http"
    "time"
)
func handler(w http.ResponseWriter, r *http.Request) {
    rand.Seed(time.Now().UnixNano())
    var t *template.Template
    if rand.Intn(5) > 2 {
        t = template.Must(template.ParseFiles("./hello.html", "./content2.html"))
    } else {
        t = template.Must(template.ParseFiles("./hello.html", "./content1.html"))
    }
    t.ExecuteTemplate(w, "model", "")
}
func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```	
- 块动作
	块动作允许定义一个模板并立即使用
	```    
	{{blocking}}
	{{end}}
	```
	```	
	./hello.html
	<!-- 定义模板 -->
	{{define "model"}}
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    {{block "content" .}}
	    如果找不到就显示我
	    {{end}}
	</body>
	</html>
	{{end}}
	```
	```	
	./content1.html
	<!DOCTYPE html>
	<html lang="en">
	<head>
	    <meta charset="UTF-8">
	    <meta name="viewport" content="width=device-width, initial-scale=1.0">
	    <meta http-equiv="X-UA-Compatible" content="ie=edge">
	    <title>模板文件</title>
	</head>
	<body>
	    {{define "content"}}
	        <h1>我是content1.html 模板文件中的内容</h1>
	    {{end}}
	</body>
	</html>
	```
	```	
	./main.go
	package main
	import (
	    "html/template"
	    "math/rand"
	    "net/http"
	    "time"
	)
	func handler(w http.ResponseWriter, r *http.Request) {
	    rand.Seed(time.Now().UnixNano())
	    var t *template.Template
	    if rand.Intn(5) > 2 {
	        t = template.Must(template.ParseFiles("./hello.html", "./content1.html"))
	    } else {
	        t = template.Must(template.ParseFiles("./hello.html"))
	    }
	    t.ExecuteTemplate(w, "model", "")
	}
	func main() {
	    http.HandleFunc("/", handler)
	    http.ListenAndServe(":8080", nil)
	}
	```
