# Strings

Source code in Go is in the UTF-8 encoding. This means string literals you create will be UTF strings, and the source files themselves are UTF-8. The string type offers no such guarantees, it stores an array of bytes, but by convention it is usually UTF-8, and you should convert to Unicode at the boundaries of your app. For a detailed overview of the internal workings of strings in Go see [Strings, bytes, runes and characters in Go](https://blog.golang.org/strings) on the official Go blog.

## Declaring strings {#declaring}

Quotation marks in Go source code behave slightly differently from other languages.

If you use double quotes you will get a string, and can include escape codes, like \n for newline:

```go
// hello is a string containing newline & tab
hello := "hello\n\tworld"
```

> hello  
>    world

If you use back quotes escape codes will not be interpreted, and the string can contain raw newlines:

```go
// hello is a string with no newline
hello := `hello world\n`
```

This results in no newline:
> hello world\n

If you use single quotes, you won't create a string at all, these create a rune:

```go
hello := 'h' // a rune
```

> h

This is why if you try to define a string with single quotes, you'll get an error:

```
// This is not a valid string
hello := 'hello'
```

> 'invalid character literal \(more than one character\)'

## Multi-line strings {#multi-line}

To define multi-line strings, including literal control characters like newline, use back quotes, for example:

```go
// A string containing a newline after hello
`hello
  world`
```

## Runes {#runes}

Runes in Go represent a code point in unicode, rather than a character. They may be a single character, or they may be part of a character or a modifier. They are an alias for the type `int32`.

For example the string:

Hello, 世界

Is represented by these bytes:

> \[72 101 108 108 111 44 32 228 184 150 231 149 140\]

but is represented by these runes:

> \[72 101 108 108 111 44 32 19990 30028\]

as you can see if you range over it (see below).

## Range on Strings {#range}

If you range on a string, you will receive the runes which make up the string, not the bytes. In an ASCII string every byte corresponds exactly to one rune, so ranging "hello" will return:

> 'h', 'e', 'l', 'l', 'o'

and ranging over "日本語" will return the runes, not the bytes:

> '日', '本', '語'

## Index on Strings {#indexing-strings}

Somewhat confusingly, given how ranging works, if you take the index of a string you'll get the byte at that index, not the rune:

```go
s := "日本語"
// This returns a byte at index 2
b := s[2]  
fmt.Println(b)
```

Returns the third byte, not the third rune as you might expect:

> 165

## Trimming strings {#encodings}

The strings package contains many useful functions for manipulating text, and each one is documented. Some might not do exactly what you expect from the name, for example strings.Trim does not trim a given suffix or prefix:

```go
s := "les mots, et les choses"
t := strings.Trim(s,"les ")
fmt.Println(t)
```

The result may be unexpected:

> mots, et les cho

This function takes a **cutset** string – a list of characters to trim. If you want to trim a full prefix or suffix, use strings.TrimPrefix or strings.TrimSuffix:

```go
s := "les mots, et les choses"
t := strings.TrimPrefix(s,"les ")
fmt.Println(t)
```

Which will remove the suffix or prefix provided as you would expect.

## Formatting Strings {#encodings}

The usual Printf and Sprintf variants are available in the fmt package, so that you can format strings using a [template string](https://golang.org/pkg/fmt/#hdr-Printing) and variables. The formats accepted are detailed in the strings package documentation. %v prints the value in a default format and is useful for debugging.

```go
fmt.Printf("%s at %v", "hello world", time.Now())
// Print to a string
s := fmt.Sprintf("%s at %v", "hello world", time.Now())
```

## Encodings {#encodings}

Most of the time when working with unicode strings in Go you won't have to do anything special. If you need to convert from another encoding like Windows 1252 you can use the [encoding](https://godoc.org/golang.org/x/text/encoding) package under golang.org/x/text. There are also some other utility packages for dealing with text there. The golang.org/x libraries are slightly less stable than the go standard library and are not part of the official distribution, but have many useful utilities.

## Translating encodings {#translating-encodings}

The go external package [text/encoding](https://godoc.org/golang.org/x/text/encoding) packages provide utilties for translating strings from Go's native UTF-8 to popular encodings and back (such as UTF-16, EUC-JP, GBK and Big5). Encoders and Decoders are available for working with arrays of bytes or strings.

```go
// Convert from Go's native UTF-8 to UTF-16
// using golang.org/x/text/encoding/unicode
s := "エヌガミ"

// Encode native UTF-8 to UTF-16
encoder := unicode.UTF16(unicode.LittleEndian, unicode.UseBOM).NewEncoder()
utf16, err := encoder.String(s)
if err != nil {
    fmt.Println(err)
}
fmt.Println("utf16:", utf16)
```

## Strings are immutable {#immutable}

Strings in Go are immutable, you are not allowed to change the backing bytes, and manipulating bytes directly can be dangerous  as the string _may_ contain several bytes for each character/rune.

## Don't use pointers to strings {#string-pointers}

A string value is a fixed size and [should not be passed as a pointer](https://github.com/golang/go/wiki/CodeReviewComments#pass-values).

## Strings are never nil {#zero-value}

You cannot assign nil to a string or compare nil to a string, they have a zero value of "" and can never be nil, only the zero value.

## Converting Strings to Ints {#atoi}

You can use the [strconv](https://golang.org/pkg/strconv/) package to convert Strings to other types like ints and floats and back again.

```go
 // Itoa converts an int to a string
 s := strconv.Itoa(99)
 // Atoi converts a string to an int
 i,_ := strconv.Atoi(s)
 // Print both types
 fmt.Printf("string:%s int:%d", s, i)
```

Returns

> string:99 int:99

## Parsing floats {#floats}

Floats are [imprecise ](http://floating-point-gui.de/)and rounding can be unpredictable. When parsing them from strings and working with them, you need to be aware of this.

```go
price := "655.18"
f, _ := strconv.ParseFloat(price, 64)
c := int(f * 100)
fmt.Printf("string:%s float:%f cents:%d", price, f, c)
```

> string:655.18 float:655.180000 cents:65517

If parsing a float for a currency value, consider converting the string to a value in cents first, as you will have work to do anyway to strip currency amounts and deal with missing cents. You can then parse as an integer and avoid any problems with storing it as a float.

## Reading text files {#reading-files}

The easiest way to read an entire file is with ioutil, but unless the file is small (e.g. a config file) this can use a large amount of memory:

```go
b, err := ioutil.ReadFile("path/to/file")
if err != nil {
   fmt.Println("error reading file: %s", err)
}

fmt.Printf("%s", b)
```

So consider reading large files in chunks. You can read a line-based file with [bufio.Scanner](https://golang.org/pkg/bufio/) and the Scan function:

```go
file, err := os.Open("path/to/file")
if err != nil {
    fmt.Println("error opening file: %s", err)
    return
}
defer file.Close()
scanner := bufio.NewScanner(file)
for scanner.Scan() {
    fmt.Println("line:%s", scanner.Text())
}
if err := scanner.Err(); err != nil {
    fmt.Println("error reading file: %s", err)
}
```

Also consider that files conform to io.Reader and io.Writer from the io package, and can be used directly with functions which accept an io.Reader.