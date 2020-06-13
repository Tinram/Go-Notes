
# Go Notes


## Arrays

+ after initialisation: fixed length, static
+ are values (not pointers): passed to function by value (change original array: use pointer)
+ not multi-dimensional (use arrays of arrays or slices of slices)
+ comparable using `==`

```go
var a[3] int
a[0] = 1
a[len(a)-1]                         // last element
var a[3] int = [3] int {1, 2, 3}    // array literal
a := [...] int {1, 2, 3}            // ellipsis: compiler calculates length

a := [2] string {"send", "rcvd"}
for index, value := range a {
...
}
```

&ndash; see **Slices**, **Maps**


## Benchmarking

```go
import "testing"
func BenchmarkNAME(b *testing.B) { ...
```

```bash
go test -bench=.    # or filename
```


## blank identifier _

```go
_, f := path.Split("a/b/file.ext")
```

&ndash; returns two values *dirName* and *fileName*, but *dirName* discarded into blank identifier


## Bytes

```go
'x'
```

## Casts

```go
b := []bytes("C€")
f := float64(i)
s := string(b)
u := uint(f)
```

```go
import "strconv"
s := strconv.FormatBool(true)
```

```go
strconv.ParseFloat("1.234", 64)
strconv.ParseInt("123", 0, 64)
strconv.Atoi("135")
strconv.Itoa(i)
```


## Channels

+ connections of same type between goroutines
+ avoid shared memory and mutexes
+ two principal operations: send, receive
+ usage: define channel, use goroutine passing channel reference, process message returned
+ channel connecting goroutines together &ndash; output to input = pipeline

...

### unbuffered


### buffered


## C modules

+ `go build` pre-processes with cgo

```go
/*
#include <stdlib.h>
*/
import "C"
```


## cmd-line

```go
import "os"
for i, arg := range os.Args {
	fmt.Println("arg", i, "=", arg)
```

```go
if len(os.Args) != 3 { ...
os.Args[1:]
os.Args[2]
```

```go
import "flag"
s := flagString("s", "x", "help text")
flag.Parse()
fmt.Println("val of s:", *s)

flag.Usage = "func() {
	fmt.Fprintln(os.Stderr, "txt")
```

### sub commands

```go
x := flag.NewFlagSet("clone", flag.ExitOnError)
switch os.Args[1] {
	case "clone": ...
```


## Commands

```go
import "os/exec"
cmdOut, err := exec.Command("go", "version").Output()
if err != nil {log.Fatal(err)}
fmt.Printf("%s", cmdOut)
```


## Comments

```go
/* */
//
```

## Compile

```bash
go build x.go
go run x.go
```

```bash
go build -x x.go                          # verbose
go build -gcflags -m                      # analysis
go build -ldflags="-s -w" x.go            # reduce exe size
go tool compile -help                     # compile flags
```

```bash
GOOS=linux GOARCH=386 go build x.go       # x32
GOOS=window GOARCH=amd64 go build x.go    # Win
```


## Constants

+ number, string, or bool
+ constants without type have greater precision ~256 bits

```go
const s string = "x"
```

```go
const (
	KILO = uint64(0x400)
	MEGA = uint64(0x100000)
)
```

*iota* = constant generator


```go
type WD int
const (
	Sunday WD = iota    // enums, start 0
	Monday ...)

const (
	= 1 << 10 * iota)
	kB
	MB ...)
```

## cpu

```go
import "runtime"
numCPUs := runtime.NumCPU()
```

## Crypto

```go
import "crypto"
```

+ AES
+ HMAC
+ MD5, SHA1, SHA256, SHA512
+ CSPRNG


## Databases


## Datatypes


## Debugging

### delve

```bash
go get https://github.com/go-delve/delve
dlv debug x.go
```

### gdb

```bash
gdb x
```


## Documentation

```bash
go doc <pkg>                      # display docs
go doc <pkg>.<member>.<method>
go doc -h -http :8000
```


## Encapsulation

+ encapsulated: inaccessible to clients of object (visibility)
+ name-based: unit of encapsulation is package (not type)
+ to encapsulate an object, make it a struct
+ fields of struct type visible to all code in same package


## Encoding


## Errors

+ error type `nil` if no error in execution
+ error handling occurs in callee

```go
if err != nil {
	fmt.Println(err)
	// or perhaps a panic() ...
```

`recover()` can be used to catch/intercept a panic in a deferred function

```go
import "errors"
// provides functions to manipulate errors
```


## exit

```go
os.Exit(1)
```

## Export

```bash
export GOPATH=$HOME/golang
```


## Files


## Format

```bash
gofmt <file>            # to stdout
      -d diff
      -w (re)write file
```

style:

+ tabs
+ CamelCase variables
+ Caps public visibility
+ filenames: file_format.go


## Functions

+ first class values
+ Go stack is dynamic, ~1GB: suited for recursion

### example

```go
func f(x, y float64) float64 {
	...
	return x, y
}
```

### anonymous / literal

```go
func() {...}
```
+ defined at point of use, access to entire lexical environment
+ enables creation of generators


### arguments

+ passed by value
+ use pointers to pass args by reference (updates / size)
+ no arg defaults, or args by name


### bootstrap / auto exec

```go
func init() {...}
```


### defer

+ deferred function executed when containing function exits
+ args for defer evaluated when defer statement is evaluated (not when function executes)
+ multiple defers executed in reverse order


### multiple returns

```go
func f() (int, string) {
	return i, s
}
q, p: = f()
```


### receiver

```go
type day string
func (d day) printDay() {
	fmt.Println(d)
}
var d day = "Monday"
d.printDay()
```


### recursive

```go
func f() {
	...
	return f()
```


### variadic

```go
func f (vals ...int) int {
```


### named return values

(aka *bare return*)

```go
func f() (x, y string) {
	x = ...
	y = ...
	return
}
```


## Goroutines


## gotchas


## goto

+ primarily for machine implementation

```go
goto E
E:
```

## hash table

```go
m := map[string]string {
	"x": "a",
	"y": "b",
}

m["x"] = "c"
```


## Help

```bash
go env
go version
go doc <pkg>
```


## Images

```go
import(
	"image"
	"image/color"
	"image/png"
)
```


## Import


## Input


## Install


## Interfaces


## JSON


## Label

```go
x:
... loop ...
	continue x
```


## Lifetimes

+ range of time in execution that variable can be referenced (reachability)
    + unreachable -> garbage collected
+ at package level
+ run-time; *cf* scope = compile-time


## Links


## Lint

```bash
golint x.go
```

[Go Critic](github.com/go-critic/go-critic)


## Lock


## Logging


## Loops


## Maps


## Methods


## misc


## Net


## null byte

```go
"\x00" // string
```


## Numbers

```go
import "math/rand"

rand.Int()
rand.Float64() * 5
rand.Seed(time.Now().Unix())

randNum := rand.Int() % len(a)

fmt.Sprintf("%s took %.5f seconds", url, t)
```

### complex

```go
import "math/cmplx"
```


## Packages

```bash
go list <pkg>
```

```bash
go vet
go fix
```


## Pointers

```go
&x                    // address of x
*int                  // pointer to int

score := 32
p := &score           // pointer assigned address
fmt.Println(p)        // prints address
*p = 44               // pointer dereferenced with value
fmt.Println(score)    // updated score

func f (x *int)
fmt.Println(*x)
```


## Print

```go
import "fmt"

fmt.Println()
fmt.Printf("txt is %v", s)    // %v default
fmt.FPrintf()
fmt.Sprintf()
```

```go
log.Printf()                 // timestamp
%+v                          // field names within struct
```

## Printf

```go
%x    hex
%t    boolean
%T    type
```


## Profiling

```bash
go tool pprof
```

```bash
go test -cpuprofile=cpu.log
go test -blockprofile=bp.log
go test -memprofile=mem.log

go tool pprof -text -nodecount=10 ./x cpu.log
```

----


```bash
go get github.com/pkg/profile
```

```go
import "github.com/pkg/profile"
defer profile.Start().Stop()
...
defer profile.Start(profile.MemProfile).Stop()
```

[Profiling Go Programs](https://blog.golang.org/pprof)


## Race Detection

```bash
go test -race x
go run -race x.go
```

[Go Race Detector](https://blog.golang.org/race-detector)


## Reflection

```go
import "reflect"
reflect.DeepEqual(x, y)
```


## Regex


## Resources


## Slices


## Sorting

```go
sort.Strings(sl)
```


## Strings


## Structs


## Swap

```go
i,j = j,i
```


## Switch

```go
switch result.ResponseCode {
	case 200, 203:
	...
	default:
	...
}
```

(no break keyword, else use `fallthrough`)


### Type Switch

```go
switch x.(type) {
	case int, uint: ...
```

bind:

```go
switch x := x.(type) {...}
```


## Testing

+ x.go >> x_test.go
+ compare results from the actual functions with known expected values
+ general fail conditions: different to that expected
+ got/want test pattern &ndash; test that got/want variables are the same in test function

```go
import "testing"
func TestNAME(t *Testing.T) {
	...
	t.Fatalf("msg")
	t.Errorf("expecting 4, got %f", actual)
```

```bash
go test -v

go test -run=coverage -coverprofile=c.out <x>
        -cover <x>
        -race <x>
```

```bash
go tool cover -html=c.out
```

## Time

```go
import "time"

time.Now()
time.Parse(time.<format>)
time.Sleep(time.Second * n)
time.Sleep(time.Millisecond * n)
log.SetFlags(log.Ltime)                 // format log output hh:mm:ss
time.AfterFunc(10 * time.Second, <fn>)
```

### Timer

```go
start := time.Now()
...
fmt.Printf("\nDone.\nElapsed: %v\n", time.Since(start))
```

### Ticker

```go
ticker := time.NewTicker(time.Millisecond * 100)

go func() {
	for {
		<-ticker.C
		fmt.Print(".")
	}
}()

ticker.Stop()
```

```go
c := time.Tick(5 * time.Second)
for t := range c {
	fmt.Printf("%v\n", t)
}
```

### Timeout

```go
for {
	select {
		case <-time.After(2 * time.Second): ...
```


## Typed Primitives

+ perhaps must useful for errors:

```go
type XErr error
```


## Type Embedding

+ allows inclusion of anonymous field in struct:

```go
type SteeringWheel struct {}
type Sedan struct {
	SteeringWheel            // field has no name
}
```

&ndash; allows Sedan to access all of SteeringWheel’s methods:

```go
func (s *SteeringWheel) Turn() { ... }

sedan := Sedan {
	SteeringWheel{},
}

sedan.Turn()
```


## Unsafe

+ low-level access outside Go environment safety net

```go
import "unsafe"
unsafe.Sizeof()
unsafe.Pointer()
```


## Variables

```go
var s string = "txt"

s := "txt"            // short var assignment, compiler infers type (in functions only, not package level)
```

+ **_** blank identifier: syntax requires var name, but logic does not
+ block level scope: braces - inner variable can access outer variables, but not vice versa
+ unassigned values: `string ""`


## Visibility

+ uppercase first character of name = *exported* &ndash; visible and accessible outside of its own package


## Wait Group

```go
import "sync"
sync.WaitGroup // blocks until a specified number of goroutines have finished
```
