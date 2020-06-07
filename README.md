## Developer notes

In 2004, "developer notes" has been the name of my first personal blog, initially used as a repository for my computer science thesis. One year later, with >150 posts, developer-notes has become a resource with tens of public comments and insights a day before being discontinued. Now I need something similar: a public repo where I can collect and organize snippets, documentation and PL stuff. I like the one-file blog format, where each topic is a post with a title, a description, the code and an explanation. 

<hr />

### #go - Template, passing value

You define a template from an HTML file, executing it passing data to it. Inside template, you define a variable in order to use a named label instead of the dot element {{ . }}

The html file is:
{% raw %}
```
{{ $valuePassed := . }}
The number is {{ $valuePassed }}
```
{% endraw %}

The golang code is:
```golang 
var tpl *template.Template

func main() {
	tpl = template.Must(template.ParseFiles("index.html"))

	err := tpl.ExecuteTemplate(os.Stdout, "index.html", 10)

	if err != nil {
		log.Println(err)
	}
}

```
Explanation:
1. create a template from an HTML file, parsing it w/ ParseFiles, then creating it w/ Must
2. load the template sending to a io.Writer, passing the template and the variable (that points to {{.}})


<hr />

### #go - io.Writer interface

As you know, the Writer interface is one of the most widely used. 
It's based on one Write method, with a simple signature:

```golang
type Writer interface {
    Write(p []byte) (n int, err error)
}
```
You find the Writer implementation in pattern like this

```golang
var b bytes.Buffer
fmt.Fprintf(&b, "Size: %v", 100)
fmt.Println(b.String()) // Output: 100
```
Explanation:
1. bytes.Buffer has the Write() method. Therefore it implements the io.Writer interface
2. fmt.Fprintf parameter type must be a Writer interface

Another case is the io.WriteString method:

```golang
f, err := os.Open("some_file.txt")
if err != nil {
   doSome(err)
}

defer f.Close()

io.WriteString(f, "a string to write in file")
```
Explanation: 
1. f is \*File, which implements the Writer interface
2. handle error
3. io.WriteString takes in input a io.Writer type. f respects this contract 


