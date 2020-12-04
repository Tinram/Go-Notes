
# Go Notes


[Arrays](#arrays)  
[Benchmarking](#benchmarking)  
[Casts](#casts)  
[Channels](#channels)  
[cmd-line](#cmd-line)  
[Compile](#compile)  
[Constants](#constants)  
[Crypto](#crypto)  
[Datatypes](#datatypes)  
[Debugging](#debugging)  
[Documentation](#documentation)  
[Errors](#errors)  
[Functions](#functions)  
[Goroutines](#goroutines)  
[Installation](#installation)  
[Interfaces](#interfaces)  
[Loops](#loops)  
[Maps](#maps)  
[Methods](#methods)  
[Numbers](#numbers)  
[Packages](#packages)  
[Pointers](#pointers)  
[Profiling](#profiling)  
[Regex](#regex)  
[Slices](#slices)  
[Strings](#strings)  
[Structs](#structs)  
[Testing](#testing)  
[Time](#time)  
[Variables](#variables)


<a id="arrays"></a>
## Arrays

+ fixed length, static
+ values (not pointers): passed to functions by value (else use pointer)
+ not multi-dimensional (use array of arrays, or slice of slices)
+ comparable using `==`


```go
var a[3] int                        // 3 ints, initialised to zero
a[0] = 1
a[len(a)-1]                         // last element
var a[3] int = {1, 2, 3}            // literal
a := [...] int {1, 2, 3}            // ellipsis: compiler calculates length
a2 := a                             // copy a

a := [2] string {"send", "rcvd"}
for index, value := range a { ...
```

&ndash; see [Maps](#maps), [Slices](#slices)


<a id="benchmarking"></a>
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


<a id="casts"></a>
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

<a id="channels"></a>
## Channels

+ connection / conduit of same datatype between goroutines
+ channels avoid shared memory and mutexes
+ *send* and *receive* are main operations
+ synchronous (*unbuffered*) or asynchronous (*buffered*)
+ usage pattern: define channel, use goroutine passing channel reference, process message returned
+ channel connecting goroutines together &ndash; output to input = pipeline


### unbuffered

```go
c := make(chan string)    // create channel, datatype string
c <- "txt"                // send "txt" to c
msg := <-c                // receive from c
```

+ blocks immediately after send operation, until item received
+ stronger sync guarantees than a buffered channel: every send with corresponding receive


### buffered

```go
c := make(chan string, 2)    // 2 messages
c <- "x"
c <- "y"
close(c)                     // no more values to be sent
receiver(c)                  // blocking action until complete
```

+ can receive up to *n* items after which send operations will block until channel is drained by at least *n - 1* items
+ if channel is full, goroutine blocked until space available; sync operations decoupled
+ when `main()` executes, `<–c`, waits for value to be sent
+ when goroutine function executes `c <– value`, waits for receiver
+ both sender and receiver must be ready to communicate, otherwise wait &ndash; channels both communicate and synchronise, yet even if buffered, are blocking &ndash; however, explicit non-blocking channels are created using `select`


### deadlocks

+ strategy to avoid deadlock (unbuffered or buffered): place send operations in own goroutine and avoid blocking `main()`

```go
func main() {
	c := make(chan int)
	go func() {
		c <- 5        // send 5
	}()
	fmt.Println(<- c)    // receive 5
}
```


### closed

+ closed channel is not testable

but:

```go
x, ok := <- c
if !ok {break}    // channel was closed and drained
```

more common:
```go
for x ... {c <- x}
close(c)
```

+ only necessary to close channel when it's important to tell receiving goroutine that all data is sent


### function arguments

```go
fn x (m <- chan string)    // read-only within function
fn x (m chan <- string)    // write-only
fn x (m chan string)       // read-write
```

... cannot receive from each channel &ndash; whichever operation tried first will block until completion: need multiplex = `select`


### select

+ creates series of receivers / conditionals for all channels
+ makes a goroutine wait on multiple communication operations
+ `select` blocks until:
    + one of the cases is matched and can be executed
    + if multiple cases are ready: chooses random case
    + if no cases can run and `default` present, executes `default` clause

```go
select {
	case msg1 := <-ch1: ...
	case msg2 := <-ch2: ...
	case <-time.After(500 * time.Millisecond):    // timeout if no messages received
	default: ...
```


### quit pattern

+ timeout
+ 'kill switch':
```go
for {
	select {
		case <- stop: ...
```


### misc

```go
c := make(chan chan string)    // channel of string channels
```


## C modules

+ `go build` pre-processes with *cgo*

```go
/*
#include <stdlib.h>
*/
import "C"
```


<a id="cmd-line"></a>
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


<a id="compile"></a>
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


<a id="constants"></a>
## Constants

+ number, string, or boolean
+ constants without type have greater precision: ~256 bits

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
	Sunday WD = iota    // enums, start at 0
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


<a id="crypto"></a>
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
[x/crypto](https://godoc.org/golang.org/x/crypto)


<a id="datatypes"></a>
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
byte (uint8) | | 0
int, int8, int16, int32, int64 | | 0
uint, uint8, uint16, uint32, uint64, uintptr | | 0
float32, float64, complex64, complex128 | | 0.0
string | | ""
rune (int32, holds Unicode char) | | 0
error | |
func, pointer, slice, map, channel | | nil


### reflection

```go
import "reflect"
fmt.Println(reflect.TypeOf(x))
```

### constants

true, false, iota, nil


<a id="debugging"></a>
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


<a id="documentation"></a>
## Documentation

```bash
go doc <pkg>                      # display docs
go doc <pkg>.<member>.<method>

godoc -http :8000
```


## Encapsulation

+ encapsulated: inaccessible to clients of object (visibility)
+ name-based: unit of encapsulation is package (not type)
+ to encapsulate an object, make it a struct
+ fields of struct type visible to all code in the same package


<a id="errors"></a>
## Errors

+ error type `nil` if no error in execution
+ error handling occurs in callee

```go
if err != nil {
	fmt.Println(err)
	// or perhaps a panic() ...
```

`recover()` can be used to catch / intercept a panic in a deferred function

```go
import "errors"
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

+ tab indentation
+ CamelCase variables
+ Caps = public visibility
+ filenames: file_format.go


<a id="functions"></a>
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
+ use pointers to pass arguments by reference (updates / size)
+ no argument defaults, or arguments by name


### bootstrap / auto exec

```go
func init() {...}
```


### defer

+ deferred function executed when containing function exits
+ arguments for `defer` evaluated when `defer` statement is evaluated (not when function executes)
+ multiple defers are stacked, and so executed in reverse order (LIFO)


### multiple returns

```go
func f() (int, string) {
	return i, s
}
q, p: = f()
```

### errors

+ for functions that can fail, by convention last argument should be error object


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

(aka *bare / naked return*)

```go
func f(i int) (x, y int) {
	x = i * 2
	y = x - i
	return
}
```


<a id="goroutines"></a>
## Goroutines

+ lightweight thread managed by Go runtime
+ concurrency &ndash; avoids blocking

```go
go <slowfunc()>
go func() { ...
```

+ not garbage collected, must exit on their own
+ unbounded parallelism of goroutines &ndash; always hits a limit: OS file descriptors, DNS etc &ndash; strategies required
+ suits order of events unknown
+ beware closures in goroutines &ndash; variables become shared

+ all anonymous goroutines functions have final `()`
+ goroutine function literal with explicit parameter in end brackets: avoids problem of loop variable capture / out-of-sync

+ no direct way to wait until a goroutine has finished
+ no way for one goroutine to terminate another directly (else would lead all shared variables in undefined states)
    + solutions:
        + broadcast mechanism: trigger, drain channels
        + inner goroutine reports completion to outer goroutine by sending event on shared channel

+ data race occurs when two goroutines access the same variable concurrently and at least one of the accesses is a write
    + data race avoidance:
        + read-only variables
        + variable confined to single goroutine, channels drive updates
        + mutexes &ndash; sync package
        ```go
		import sync
		var m sync.Mutex
		m.Lock()
		...
		m.Unlock()
		```

+ counter that can be safely changed from multiple goroutines: `sync.WaitGroup`


## gotchas

+ call any method on zero value = panic
+ no immutable data structures
+ channels do not prevent race conditions because Go lacks immutability
+ [gotchas and mistakes](devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)


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
import _ "xyz"    // aliasing package qualifier to _ so none of its exported names are visible
import . "xyz"    // imports package into your namespace: get rid of fmt prefix etc
```


## Input

```go
var input string
fmt.Scanln(&input)
```


<a id="installation"></a>
## Installation

### standalone

[https://golang.org/doc/install](https://golang.org/doc/install)

```bash
tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
$HOME/.profile >> export PATH=$PATH:/usr/local/go/bin
```

### golang repo

*avoid outdated distro repo:*

```bash
sudo add-apt-repository ppa:longsleep/golang-backports
sudo apt-get update
sudo apt-get install golang-go
```


<a id="interfaces"></a>
## Interfaces

+ specifies set of method signatures
+ calling method on an interface value executes the method of the same name on its underlying type

```go
type I interface {
	M()
}

type T struct {
	S string
}

// type T implements the interface I (not explicitly declared)
func (t T) M() {
	fmt.Println(t.S)
}
```


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

log.Printf("txt")    // terminal
log.Fatal("txt")

f, err := os.OpenFile("f.log", os.O_APPEND|os.O_CREATE|os.O_RDWR, 0666)
if err != nil {
	log.Fatal(err)
}
defer f.Close()
log.SetOutput(f)
log.Printf("txt")
```


<a id="loops"></a>
## Loops

```go
for i := 0; i < 10; i++ {       // traditional

for i < 10 {                    // while
for {}                          // infinite

// range iterates over slice or map:
for _, u := range sl {

for v := range x {              // index
for _, v := range x {           // value

nums := [] int {1, 2, 3}
for i, n := range nums {

b := []string{"a", "b", "c"}
a = append(a, b...)             // variadic argument expands b to all members: avoids for...range loop

continue
break
```

+ only postfix i++, no prefix
+ scope: beware of 'iteration variable capture' &ndash; inner var copy needed for correct operation


<a id="maps"></a>
## Maps

+ set of key / value pairs
    + key: any type that can be compared with `==`
    + value: any type
+ unordered 'associative array'
+ accessed by key
+ dynamic
+ elements not addressable (slice elements are) &ndash; use temporary variable as fix


```go
m := map[string] int{"k":1}         // key = string, value = int
// else: m["k"] = 1
fmt.Println(m["k"])

delete(m, "k")
a, ok := m["k"]                     // test if key "k" present and retrieve if so
if !ok { ...                        // ok = boolean

sk = strings: sort.Strings(m)

for k, v := range m {               // iterate over map content
```

### map literal

```go
ratings := map[string][]int {
	"men":   {32, 55},
	"women": {44, 42},
}

map [T} bool    // set
```


<a id="methods"></a>
## Methods

+ define methods on datatypes
+ a function with a receiver argument
+ receiver argument appears before method name
+ no `this` or `self` for receiver
+ methods with pointer receivers can take either a pointer or a value (unlike functions)
+ can define methods for any datatypes e.g. slices (Go is rather unusual)
+ method sets: multiple methods associated with the struct


```go
type Movie struct { ... }

func (m *Movie) summary() string { ...    // makes summary method available for any Movie instance
        // ^ receiver - by pointer or val
        // pointer can modify elements within original struct: use to modify original initialisation of struct

```
+ ideal usage: sphere struct, surface area and volume methods


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
	ford := &car {make: "Ford", model: "F150",}
	ford.drive()
}
```

----


```go
type Point struct {...}
func (p Point) Distance (q Point) ...    // traditional: function Distance(p, q, Point)
p.Distance(q)
// p calls method's receiver
distance := Point.Distance               // method expression
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

+ make sure both mutex, and guarded variables, are not exported

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
"\x00"    // string
```


<a id="numbers"></a>
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


<a id="packages"></a>
## Packages

```go
import (
	"fmt",
	"math"
	_"xyz"    // ignores, to compile
)
```

```bash
go list <pkg>
```

```bash
go vet
go fix
go get -u    # ensure all pkgs / deps (including local) are latest version
```

### 3rd Party

```bash
go get <url>
```

```go
import ("<url>")
```

```bash
# ~/.bash_profile:
export GOPATH=$HOME/dir/
source ~/.bash_profile
```

### Update

```bash
# project folder
go get -u

# specific
go get -u <url>
```

### Create

```go
package mine
import ("testing")
func MyFunc() {    // public
func x() {         // private
```
```bash
go test    # test package
```

```go
package httpsimple
import ...

//

package main

import(
	"fmt"
	"httpsimple"
)

httpsimple.Get(url)
```

[golang.org/pkg](https://golang.org/pkg)


<a id="pointers"></a>
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


<a id="profiling"></a>
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


<a id="regex"></a>
## Regex

```go
import "regexp"

x := regexp.MatchString("p([a-z]+)ch", "peach")

re := regexp.MustCompile(`<a\s[^>]*href=\"([^\"]*)\"[^>]*>(.*)<\/a>`)    // panics if cannot compile
fmt.Println(re.MatchString(s))

r, _ := regexp.Compile("p([a-z]+)ch")    // error if cannot compile
r.MatchString("peach"))
r.FindStringIndex("peach punch"))
r.FindAllString("peach punch pinch", -1)
r.ReplaceAllString("a peach", "<fruit>")
```


## Resources

+ open files and network connections should be closed explicitly


<a id="slices"></a>
## Slices

+ dynamic size, variable-length
+ can contain any datatype (with all elements of  same datatype)
+ basically pointers to underlying array &ndash; like an ArrayList
    + multiple slices can share same underlying array
+ slices not comparable
    + but `bytes.Equal` for byte slices
+ 3 components: pointer, length, capacity
    + pointer: first element if (array) that is reachable
    + length: number of slice elements, which cannot exceed capacity
    + slicing beyond capacity = panic
    + slicing beyond len(sl) = extends slice

**create:**

```go
sl := make([]T, <len> [, cap])
sl := []T{}
bs := []byte("txt")
```

```go
len(sl)
cap(sl)           // capacity
append(sl, x)     // append and delete; append() is variadic: multiple additions
copy(dst, src)    // element from one to another of same type

var sl = [] string                    // nil slice
sl := make([] int, 0)                 // empty slice: len(sl) == 0 test empty
sl := []int {0, 1, 2, 3}              // literal: like array literal, but no size given
m := []string {1: "Jan", ...}         // => 0
sl := make([]byte, 5)                 // 5 bytes

sl := make([]string, 1)
sl[0] = "txt"
sl = append(sl, "txt2" [, "txt3"])    // append() is variadic: multiple additions

sl.append(sl[:2], sl[2+1:...)         // delete element 2

summer := months[5:8]                 // from existing slice (or array)
c := append(a, b...)                  // concat slices a and b

a:= [2]string {"Лайка", "Белка"}      // array
sl := a[:]                            // whole slice of a

sl[i:j]                               // i to j - 1, if j omitted = len(s)
sl[:]                                 // whole
s[:5]                                 // extended slice (within capacity)

x := 2; y := 4
table := make([][]int, x)
for i:= range table {table[i] = make([]int, y)}

copy(sl2, sl)                         // same type
copy(sl2, sl[1:])                     // element 1

for _, v := range sl {
	...
}
```


## Sorting

```go
sort.Strings(sl)
```


<a id="strings"></a>
## Strings

+ immutable: read-only byte slices -> leading to cheap actions for copying, substrings
+ UTF-8
+ no string indexing
+ each character of type 'rune' = int32
+ string len() = number bytes (not runes) &ndash; is array of runes / bytes (length of runes versus bytes differs with multibytes)
+ main packages: *bytes*, *strings*, *strconv*, *unicode*


| | | description
:- | :-: | :- |
`"..."` | | string literal, can contain escape sequences `\t`, `\xhh`
`...` | | raw string literal, can be multi-line, no escape sequences
`\uhhhh` | | 16-bit Unicode value
`'\x41'` | | rune < 256 value
`rune` | | int32, Unicode codepoint

```go
var x string                 // defaults to ""

s := `This is
multiline`                   // multiline (including whitespace)

+ / +=                       // concat
len(s)                       // number of bytes, use "unicode/utf8" >> utf8.RuneCountInString(data)

s[0] = "x"                   // indexing = 104 (accesses byte)
s[0:5] - 0 to 5 chars
fmt.Println("%q", s[0])

strconv.Itoa(x)

strings.Contains(h, n)
strings.Index(h, n)

strings.Count()

strings.HasPrefix()          // from start
strings.HasSuffix()          // from end

strings.Join()
strings.Split()

strings.ToLower()

strings.TrimSpace()

strings.ReplaceAll

```

**index**

```go
s := err.Error()
if strings.Index(s, "no host") > -1 { ...
```

Index returns byte value, not character  
golang.org/x/exp/utf8string has `At()`


```go
var s int = strings.Index(link, strings.Join(u, ""))
b := strings.Contains(h, n)
```

**string-indexing hack**
```go
x := "text"
xbytes := []byte(x)
xbytes[0] = 'T'
fmt.Println(string(xbytes))    // but best to use runes
```

**runes &ndash; multi-byte**
```go
myString := "test test"
r := []rune(myString)
s := string(r[0:6])
```

**buffering**
```go
var buffer bytes.Buffer
for i := 0; i < 500; i++ {
	buffer.WriteString("z")
}
fmt.Println(buffer.String())
```

**string builder**
```go
import("bytes")
var b bytes.Buffer
for i := 0; i < 1000; i++ {
	b.WriteString(randString())
}
fmt.Println(b.String())
```

**int to string**
```go
fmt.Sprintf("%d", i)
// or
import("strconv")
s := strconv.Itoa(i)
```

**string to byte slice**
```go
b := []byte(s)
```

**byte slice to string**
```go
s := string(b)
```


<a id="structs"></a>
## Structs

+ collection of fields
+ dynamic values
+ can be nested
+ data fields in lowercase will not be encoded
+ may contain mixture of unexported and exported fields
+ can be used as a key of map
+ data structure to JSON = marshalling -> byte slice

```go
type Movie struct {    // uppercase names and elements = public
	Name string    // no comma
	Rating float32
}

m := Movie {
	Name: "Citizen Kane",
	Rating: 9.8,   // comma required
}

m := new(Movie)
m.Name = "Metropolis"
c := Movie{Name: "x", Rating: 0,}
```

----

```go
type Point struct {X, Y int}    // struct literal, type 1
p := Point {1, 2}
p := &Point {1, 2} = p := new(Point); *p = Point {1, 2}

a := gif.GIF {loopCount: n}    // type 2
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


<a id="testing"></a>
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


<a id="time"></a>
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
	SteeringWheel        // field has no name
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


<a id="variables"></a>
## Variables

```go
var s string = "txt"

s := "txt"            // short assignment, compiler infers type
```

+ `var` can be defined at both package and function level; short assignment only at function level
+ **_** blank identifier: syntax requires variable name, but logic does not
+ block level scope: braces &ndash; inner variable can access outer variables, but not vice versa
+ unassigned values: `string ""`


## Visibility

+ uppercase first character of name = *exported*: visible and accessible outside of its own package


## Wait Group

+ waits until a specified number of goroutines have finished before resuming program execution
+ works best when there is no need of results from goroutines

```go
import "sync"
var wg sync.WaitGroup
wg.Add(1)
go func() {
	...
	wg.Done()
}
wg.Wait()
```
