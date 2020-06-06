## Developer notes

In 2004, "developer notes" has been the name of my first personal blog, initially used as a repository for snippets, PL documentations, and other stuff related to computer science. One year later, with >150 posts, developer-notes has become a resource with tens of comments and insights a day. Now I need something similar: a public blog with technical documentation, easy to search, easy to organize and populate. 
Each post has a title, a description, the code, an explanation and a series of tags in order to keep it easy to search. 

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


