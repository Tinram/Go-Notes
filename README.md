
# Go Notes


## Arrays


## Benchmarking


## blank identifier _


## Bytes


## Casts

```go
b := []bytes("C€")
f := float64(i)
s := string(b)
u := uint(f)
```

```go
import ("strconv")
s := strconv.FormatBool(true)
```

```go
strconv.ParseFloat("1.234", 64)
strconv.ParseInt("123", 0, 64)
strconv.Atoi("135")
strconv.Itoa(i)
```


## Channels


### unbuffered


#### buffered


## cmd-line


## Commands


## Comments

```go
/* */
//
```

## Compile


## Constants

+ number, string, or bool
+ constants without type have greater precision ~ 256 bits

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


## Typed Primitives


## Type Embedding


## Variables


## Visibility


## Wait Group
