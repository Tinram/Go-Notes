
# Go Notes


## Arrays

+ after initialisation: fixed length, static
+ are values (not pointers): passed to function by value (change original array: use pointer)
+ not multi-dimensional (use arrays of arrays or slices of slices)
+ comparable using `==`


```go
var a[3] int                        // 3 ints, initialised to zero
a[0] = 1
a[len(a)-1]                         // last element
var a[3] int = [3] int {1, 2, 3}    // array literal
a := [...] int {1, 2, 3}            // ellipsis: compiler calculates length
a2 := a                             // copy

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


### unbuffered

```go
c := make(chan string)    // create channel of type string
c <- "txt"                // send to c
msg := <-c                // receive from c
```

+ blocks immediately after send operation until item received
+ stronger sync guarantees than with buffered channel &ndash; every send with corresponding receive


### buffered

```go
c := make(chan string, 2)    // 2 messages
c <- "x"
c <- "y"
close(c)
receiver(c)                  // blocking action until complete
```

+ can receive up to N items after which send operations will block until channel is drained by at least n-1 item
+ if channel full, goroutine blocked until space available; sync operations decoupled
+ when main() executes `<–c`, waits for value to be sent
+ when goroutine function executes `c <– value`, waits for receiver
+ both sender and receiver must be ready to communicate, otherwise wait &ndash; channels both communicate and synchronise, yet even if buffered, are blocking &ndash; however, explicit non-blocking channels are created using `select`


### deadlocks

+ strategy to avoid deadlock (unbuffered or buffered): place send operations in own goroutine and avoid blocking main()

```go
func main() {
	c := make(chan int)
	go func() {
		c <- 5  // send 5
	}()
	fmt.Println(<- c) // receive 5
}
```


### closed

+ closed channel is not testable

but:

```go
x, ok := <- c
if !ok {break{    // channel was closed and drained
```

more common:
```go
for x ... {c <- x}
close(c)
```

+ only necessary to close channel when important to tell receiving goroutine that all data is sent


### function arguments

```go
fn x (m <- chan string)    // read-only within function
fn x (m chan <- string)    // write-only
fn x (m chan string)       // read-write
```


### select

+ cannot receive from each channel = whichever operation tried first will block until completion: need multiplex = `select`
+ creates series of receivers / conditionals for channels
+ makes goroutine wait on multiple communication operations
+ blocks until one of its cases can run, then executes that case; chooses one at random if multiple cases are ready
+ for where only first goroutine returned is acted upon

```go
select {
	case msg1 := <-ch1: ...
	case msg2 := <-ch2: ...
	case <-time.After(500 * time.Millisecond):    // timeout if no messages received
	default: ...
```


### quit pattern

+ timeout
+ kill switch:
```go
for {
	select {
		case <- stop: ...
```


## C modules

+ `go build` pre-processes with *cgo*

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
+ RSA
+ x509

also:  
[x/crypto](golang.org/x/crypto)


## Datatypes

4 categories:

+ **basic**
+ **aggregate**
    + arrays, structs
+ **reference**
    + channels, functions, maps, channels, pointers, slices
+ **interface**


datatype | | zero value
:- | :-: | -: |
bool | | false
byte (uint8) | |
int, int8, int16, int32, int64 | | 0
uint, uint8, uint16, uint32, uint64, uintptr | | 0
float32, float64, complex64, complex128 | | 0.0
string | | ""
rune (int32, Unicode char) | |
error | |
func, pointer, slice, map, channel | | nil


### reflection

```go
import "reflect"
fmt.Println(reflect.TypeOf(x))
```

### constants

true, false, iota, nil


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
// provides many functions to manipulate errors

errors.Wrap(err, "msg")

errors.Cause(err).(type)
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

```go
import "os"
f, _ := os.Create("x.txt")
fmt.Fprint(f, "msg")
f.Close()

f, err := os.Open("x.txt")
if err != nil { ... }
f.Close()
```

```go
import "io/ioutil"
f, err := ioutil.ReadFile("f.dat")
if err != nil ( ...
```


## Format

```bash
gofmt <file>            # to stdout
      -d diff
      -w (re)write file
```

### style

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

### errors

By convention, for functions that can fail, last argument should be error object.


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

...


## gotchas

...


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

```go
import _ "xyz"     // aliasing package qualifier to _ so none of its exported names are visible
import . "xyz"     // imports package into your namespace: get rid of fmt prefix etc
```


## Input

```go
var input string
fmt.Scanln(&input)
```


## Install

### standalone

[https://golang.org/doc/install](https://golang.org/doc/install)

```bash
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
$HOME/.profile >> export PATH=$PATH:/usr/local/go/bin
```

### install

*avoid outdated repo:*

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```


## Interfaces

...


## JSON

```go
json.Marshal()
json.Unmarshal()
```
```go
textBytes := []byte(json)
people := people{}
err := json.Unmarshal(textBytes, &people)
```

```go
values := map[string]string{"x": "x", "y": "y"}
jsonValue, _ := json.Marshal(values)
resp, err := client.Post(url, "application/json", bytes.NewBuffer(jsonValue))
```


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

...



## Lint

```bash
golint x.go
```

[Go Critic](github.com/go-critic/go-critic)


## Lock

```go
import "sync"

var hits struct {
	sync.Mutex
	n int
}

hits.Lock()
hits.n++
hits.Unlock()
```


## Logging

```go
import "log"

log.Printf("txt") // terminal
log.Fatal("txt")

f, err := os.OpenFile("f.log", os.O_APPEND|os.O_CREATE|os.O_RDWR, 0666)
if err != nil {
	log.Fatal(err)
}
defer f.Close()
log.SetOutput(f)
log.Printf("txt")
```


## Loops

```go
for <cond> {                    // while
for {}                          // infinite

for v := range x {              // index
for _, v := range x {           //value

for i < 10
for i := 0; i < 10; i++ {

nums := [] int {1, 2, 3}
for i,n := range nums {

for _, u := range sl {          // iterate over slice

b := []string{"a", "b", "c"}
a = append(a, b...)             // variadic argument expands b to all members - avoids for...range loop

continue
break
```

+ only postfix i++, no prefix
+ scope: beware of 'iteration variable capture' &ndash' inner var copy needed for correct operation


## Maps

...


## Methods

+ are really functions that require a receiver
+ no `this` or `self` for receiver
+ receiver argument appears before method name
+ can define methods for any types e.g. slices (Go is rather unusual)
+ method sets: multiple methods associated with the struct


```go
type Movie struct { ... }

func (m *Movie summary() string { ...    // makes summary method available for any Movie instance
         // ^ receiver - by pointer or val
		 // pointer can modify elems within original struct: use to modify original init of struct

```
+ ideal usage: sphere struct, surface area & volume methods


----


```go
type car struct {
	make,
	model string
}

func (c *car) drive() {
	fmt.Println("driving a", c.make, c.model)
}

func main() {
	ford := &car {make:  "Ford", model: "F150",}
	ford.drive()
}
```

----


```go
type Point struct {...}
func (p Point) Distance (q Point) ...    // traditional: function Distance(p, q, Point)
p.Distance(q)
// p called method's receiver
distance := Point.Distance // method expression
fmt.Prinln(distance(p, q))
```


## misc

`;` multiple statements


## Modules

```bash
go mod init <module>
```

&ndash; create a new module in the current directory (*go.mod*)


## Mutexes

+ make sure both mutex, and guarded vars, are not exported

`RLock()` &ndash; readers / shared

`sync.Once` &ndash; for initialisation / all goroutines observe variable state:

```go
var x sync.Once
x.Do(...)
```

### defer

+ ensures an unlock is not forgotten within a function and does not get in the way of a `return`

```go
	...
	defer mu.Unlock()
	return x
}
```


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

### csprng

```go
import "crypto/rand"
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

+ no pointer arithmetic


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

```go
import "regexp"

x := regexp.MatchString("p([a-z]+)ch", "peach")

re := regexp.MustCompile(`<a\s[^>]*href=\"([^\"]*)\"[^>]*>(.*)<\/a>`) // panics if cannot compile
fmt.Println(re.MatchString(s))

r, _ := regexp.Compile("p([a-z]+)ch") // error if cannot compile
r.MatchString("peach"))
r.FindStringIndex("peach punch"))
r.FindAllString("peach punch pinch", -1)
r.ReplaceAllString("a peach", "<fruit>")
```


## Resources

+ open files and network connections should be closed explicitly


## Slices

...


## Sorting

```go
sort.Strings(sl)
```


## Strings

...


## Structs

+ dynamic values
+ can be nested
+ data fields in lowercase will not be encoded
+ may contain mixture of unexported and exported fields
+ can be used as a key of map
+ data structure to JSON = marshalling -> byte slice

```go
type Movie struct {    // uppercase names and elements = public
	Name string        // no commas
	Rating float32
}

m := Movie {
	Name: "Citizen Kane",
	Rating: 9.8,      // comma required
	}

m := new(Movie)
m.Name = "Metropolis"
c := Movie{Name: "x", Rating: 0,}
```

----

```go
type Point Struct {X, Y int} // struct literal, type 1
p := Point {1, 2}
p := &Point {1, 2} = p := new(Point); *p = Point {1, 2}

a := gif.GIF {loopCount: n} // type 2
```

---

```go
type s struct {
	t string
	b string
}

a := test{t: "a", b: "body a"}
var arr []s
arr = append(arr, a)
fmt.Println(arr)

```

----

### copy

```go
s1 := s2                  // by val
s2 := &s1                 // by ref
fmt.Printf("%" + v, *s2)
```

### compare

structs of same type can be compared with `==`

default values: through constructor:

```go
type Alarm struct {
	Time string
}

func NewAlarm(time string) Alarm {
	a := Alarm {
		Time: time,
	}
	return a
}
```


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
+ block level scope: braces &ndash; inner variable can access outer variables, but not vice versa
+ unassigned values: `string ""`


## Visibility

+ uppercase first character of name = *exported* &ndash; visible and accessible outside of its own package


## Wait Group

```go
import "sync"
sync.WaitGroup // blocks until a specified number of goroutines have finished
```
