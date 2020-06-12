
# Go Notes


## Arrays

+ after initialisation &ndash; fixed length, static
+ are values (not pointers) &ndash; passed to a function by value (to change original array, use pointer )
+ not multi-dimensional &ndash; use arrays of arrays or slices of slices
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
    Sunday WD = iota // enums, start 0
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


## Encapsulation


## Encoding


## Errors


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


## Functions


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


## Links


## Lint


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


## Packages


## Pointers


## Print


## Printf


## Profiling


## Reflection

```go
import "reflect"
reflect.DeepEqual(x, y)
```


## Regex


## Resources


## Slices


## Sorting


## Strings


## Structs


## Swap

```go
i,j = j,i
```


## Switch


## Testing


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

+ low-level access outside Go env safety net

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

+ uppercase first character of name = *exported*  &ndash; visible and accessible outside of its own package


## Wait Group

```go
import "sync"
sync.WaitGroup // blocks until a specified number of goroutines have finished
```
