## Developer notes

In 2004, "developer notes" has been the name of my first personal blog, initially used as a repository for my computer science thesis. One year later, with >150 posts, developer-notes has become a resource with tens of public comments and insights a day before being discontinued. Now I need something similar: a public repo where I can collect and organize snippets, documentation and PL stuff. I like the one-file blog format, where each topic is a post with a title, a description, the code and an explanation. 

<hr />

### #python - Metaprogramming
Python's metaprogramming implementation is a -yet another- demonstration of its wonderful design. Academically, Python is a source of inspiration and beauty, even in apparently esotheric topics like that. Before explaining how M works, let's start with an introduction first: people actually don't need to use Metaprogramming on a daily basis. But a good understanding it may helps to know & appreciate how things work under the hood, because every Py3 dev is transparently using metaclasses: just think what `type` is: when you create a custom class, you're creating `type` 's instance (that is: everything is an object, classes too). So you can think at metaclasses as "class of classes", allowing you to instantiate a class, as its object, and `type` is the metaclass Py3 uses to create classes. Simple as that.
When you need to use metaclass? Not very often, actually. A typical case is when you need a high level of abstraction, endorsing "declaration over implementation", not requiring users doing other than setup-like stuff. For example when you you want to expose simple APIs, hiding complexities, logic and implementation. Look at the code I wrote here: consuming simple `sql.query("update", "table_2")` you're calling the dynamic SQL's method `update_table_2(self) -> str`, which is built via the metaclass query definition inside the dunder __new__. In a structure like that, you want your API consumer to ignore metaclass implementation, just requiring him to build its own methods toolset (update_table_1(), update_table_2(), ...) that will feed the exit point `sql.query(<action>, <target>)`.

```python
from typing import Any, Callable,  TypeAlias, Type

MSQL: TypeAlias = "MetaSQL"


class MetaSQL(type):
    query: Callable[[MSQL, str, str], str]

    def __new__(cls: Type[MSQL], name: Any, bases: Any, attrs: Any) -> MSQL:
        new_cls: MSQL = super().__new__(cls, name, bases, attrs)

        def query(self: MSQL, action: str, target: str) -> str:
            haystack: dict[str, Callable[[MSQL], str]] = {k: v for k, v in attrs.items() if k.startswith(action)}

            res: str = f"action '{action}' not found"
            needle: str = f"{action}_{target}"

            if needle in haystack:
                res = haystack[needle](self)

            return res

        new_cls.query = query

        return new_cls


class SQL(metaclass=MetaSQL):
    query: Callable[[str, str], str]

    def update_table_1(self) -> str:
        return "UPDATE table_1 ..."

    def update_table_2(self) -> str:
        return "UPDATE table_2 ..."

    def delete_table_3(self) -> str:
        return "DELETE table_3 ..."

    def create_table_4(self) -> str:
        return "CREATE table_4 ..."


sql = SQL()
a: str = sql.query("update", "table_1")
b: str = sql.query("update", "table_2")
c: str = sql.query("delete", "table_3")
d: str = sql.query("create", "table_4")
e: str = sql.query("insert", "table 5")

print(a)
print(b)
print(c)
print(d)
print(e)

# OUTPUT:
# UPDATE table_1 ...
# UPDATE table_2 ...
# DELETE table_3 ...
# CREATE table_4 ...
# action 'insert' not found
```


### #go - Generics: GcShape stenciling with dicts

If you're a #golang dev, you probably agree w/ me it's time for you to start to build your own opinion about the way `parametric polymoprhism` is implemented in go, no matter of blog authors saying "Awesome!", "Slow!" or "Awful!". Just refer to [the docs](https://github.com/golang/proposal/blob/master/design/generics-implementation-dictionaries-go1.18.md), understand how it works, then read my list of personal thoughts:

1) current G implementation is something "hybrid", a balance between compiling and runtime: is monomorphization-based, w/ virtual table support. Actually they're trying to get the best from compiler building mono-morphed "copies" for each generic type, then hitting runtime the less is possible passing types metadata in vTables. This is what they call "GcShape stenciling w/ dicts";

2) everything is grouped by its "GC shape", which is the underlying type (note: NOT type). So *pointers are grouped together, no matter of their referenced type. Yeap, Vtables to the rescue, but remember they live at runtime... and this can create some performance issue;

3) If you look at G like a code optimization, you can be disappointed, because you'll risk to move stuff from compile to run time;

4) Finally you'll deduplicate that ugly code, but keep in mind it will be duplicated back at compile time, transparently;

5) as a general rule, it can be better to avoid interface{}, just because they add a level of complexity. You can now finally avoid, as well as you can better enforcing type safety;

6) maybe it's not a good idea to rewrite your old interface-based APIs: w/ current G implementation you'll probably create a less-readable code, maybe less performant;

Cheers 🍻



### #go - 3 different ways of outputting

```golang
func main() {
	// 1
	r := strings.NewReader("hello, world!\n")

	_, err := io.Copy(os.Stdout, r)
	if err != nil {
		panic(err)
	}

	// 2
	io.WriteString(os.Stdout, "hello, world!\n")

	// 3
	ns := strings.NewReader("hello, world!\n")
	scanner := bufio.NewScanner(ns)
	scanner.Split(bufio.ScanRunes)

	for scanner.Scan() {
		fmt.Printf("%s", scanner.Text())
	}
}
```


### #go - Template, text New/Execute

text/template system needs to be defined with the New(), where you define the skeleton.
Once defined the tmpl, you need to send an object to a io.Writer. 

```golang
// Account manager
type Account struct {
	ID       int64
	Balance  float64
	Currency string
}

// DBCall mock account resume
func DBCall() Account {
	return Account{ID: 10298301, Balance: 150000, Currency: "€"}
}

func main() {
	tmpl, err := template.New("POS-result").Parse("Account: {{.ID}}\nBalance: {{.Balance}}{{.Currency}}\n")

	if err != nil {
		log.Fatalf("Terminal error on preparation: %v", err)
	}

	err = tmpl.Execute(os.Stdout, DBCall())

	if err != nil {
		log.Fatalf("Terminal error on visualization: %v", err)
	}
}

// Output
// Account: 10298301
// Balance: 150000€

```
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
// Output: The number is 10.
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


