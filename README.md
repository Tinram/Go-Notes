
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
var s string = "true"
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


## Switch


## Testing


## Time


## Typed Primitives


## Type Embedding


## Variables


## Visibility


## Wait Group
