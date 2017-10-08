# Strings

Source code in Go is in the UTF-8 encoding. This means string literals you create will be UTF strings, and the source files themselves are UTF-8. The string type offers no such guarantees, it stores an array of bytes, but by convention it is usually UTF-8, and you should convert to Unicode at the boundaries of your app. For a detailed overview of the internal workings of strings in Go see this [strings overview](https://blog.golang.org/strings) on the official Go blog.

## Declaring strings {#declaring}

Quotation marks in Go source code behave slightly differently from other languages.

If you use double quotes you will get a string, and can include escape codes, like \n for newline but not newlines:

```go
// hello is a string ending in newline
hello := "hello\n" -> "hello\n"
```

If you use back quotes escape codes will not be interpreted, and it can contain newlines:

```go
// hello is a string ending with n
hello := `hello\n` -> "hello\\n"
```

If you use single quotes, you won't create a string at all, these create a rune:

```go
// hello is a rune, not a string
hello := 'h'
```

This is why if you try to define a string with single quotes, you'll get an error:

```
'hello'
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

as you can see if you range over it.

## Range on Strings {#range}

If you range a string, you will receive the runes which make up the string, not the bytes. In an ASCII string every byte corresponds exactly to one rune, so ranging "hello" will return:

> 'h', 'e', 'l', 'l', 'o'

and ranging over "日本語" will return the runes, not the bytes:

> '日', '本', '語'

## Index on Strings {#indexing-strings}

Somewhat confusingly, given how ranging works, if you take the index of a string you'll get the byte at that index, not the rune:

```go
s := "日本語"
// This returns a byte at index 2, not the second rune as you might expect
b := s[2]  // the byte at index 2
fmt.Println(b)
```

Returns the third byte, not the third rune as you might expect:

> 165

## Trimming strings {#encodings}

The strings package contains. Since you'll be using this a lot, you should read it in its entirety at least once. Go do that now. Since you have, it'll come as no surprise that strings.Trim doesn't do quite what you might expect:

```
s := "les mots, et les choses"
t := strings.Trim(s,"les ")
fmt.Println(t)
```

The result may be unexpected:

> mots, et les cho

This function takes a **cutset** string, not a prefix or suffix. If you want to trim a full prefix or suffix, use strings.TrimPrefix and strings.TrimSuffix:

```
s := "les mots, et les choses"
t := strings.TrimPrefix(s,"les ")
fmt.Println(t)
```

## Formatting Strings {#encodings}

The usual Printf and Sprintf variants are available in the fmt package, so that you can format strings using a [template string](https://golang.org/pkg/fmt/#hdr-Printing) and variables. %v prints the value in a default format and is useful for debugging.

```go
fmt.Printf("%s at %v", "hello world", time.Now())
// Print to a string
s := fmt.Sprintf("%s at %v", "hello world", time.Now())
```

## Encodings {#encodings}

Most of the time when working with unicode strings in Go you won't have to do anything special. If you need to convert from another encoding like Windows 1252 you can use the [encoding](https://godoc.org/golang.org/x/text/encoding) package under golang.org/x/text. There are other utility packages for dealing with text there.

## Strings are immutable {#immutable}

Strings in Go are immutable, you are not allowed to change the backing bytes, and this is dangerous anyway as the string may contain several bytes for each character/rune.

## Don't use pointers to strings

A string value is a fixed size and[ should not be passed as a pointer](https://github.com/golang/go/wiki/CodeReviewComments#pass-values).

## Strings are never nil {#zero-value}

They have a zero value of "", and you cannot assign nil to a string.

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

## Parsing floats

Floats are [imprecise ](http://floating-point-gui.de/)and rounding can be unpredictable. When parsing them from strings and working with them, you need to be aware of this.

```
price := "655.18"
f, _ := strconv.ParseFloat(price, 64)
c := int(f * 100)
fmt.Printf("string:%s float:%f cents:%d", price, f, c)
```

> string:655.18 float:655.180000 cents:65517

If you're parsing a float for a currency value, you should store it as an integer cents value, so consider converting the string to a value in cents first, as you will have work to do anyway to strip currency amounts and deal with missing cents. You can then parse as an integer and avoid any problems with float calculations.

## Reading text files

You can read an entire file with ioutil, but if the file is large this will use a large amount of memory:

```go
b, err := ioutil.ReadFile("path/to/file")
if err != nil {
   fmt.Println("error reading file: %s", err)
}

fmt.Printf("%s", b)
```

Unless you are ready small files like config files, consider reading the file in chunks. The easiest way to read a line-based file is with [bufio.Scanner](https://golang.org/pkg/bufio/). You can also read the entire file into memory with ioutil, but this won't be suitable for parsing large files.

```go
file, err := os.Open("path/to/file")
if err != nil {
    fmt.Println("error opening file: %s", err)
    return
}
defer file.Close()

scanner := bufio.NewScanner(file)
for scanner.Scan() {
    // Do something with line
    fmt.Println("line:%s", scanner.Text())
}

if err := scanner.Err(); err != nil {
    fmt.Println("error reading file: %s", err)
}
```



