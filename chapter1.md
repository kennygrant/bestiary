# Strings

Source code in Go is in UTF-8 encoding. This means string literals you create will be UTF strings, and the source files themselves are UTF-8. For a detailed overview of the internal workings of strings in Go see this [strings overview](https://blog.golang.org/strings) on the official Go blog.

## Declaring strings

Quotation marks in Go source code behave slightly differently from other languages.

If you use double quotes you will get a string, and can include escape codes, like \n for newline:

```go
// hello is a string ending in newline
hello := "hello\n" -> "hello\n"
```

If you use back quotes escape codes will not be interpreted:

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

## Multi-line strings

To define multi-line strings, including literal control characters like newline, use back quotes, for example:

```go
// A string containing a newline after hello
`hello
 world`
```

## Range on Strings

If you range a string, you will receive the runes which make up the string, not the bytes. In an ASCII string every byte corresponds exactly to one rune, so ranging "hello" will return:

```
'h', 'e', 'l', 'l', 'o'
```

however ranging over "日本語" will return the runes, not the bytes:

```
'日', '本', '語'
```

## Runes

Runes in Go represent a code point in unicode, rather than a character. They may be a single character, or they may be part of a character or a modifier. They are an alias for the type `int32`.

For example the string:

> Hello, 世界

Is represented by these bytes:

```
 [72 101 108 108 111 44 32 228 184 150 231 149 140]
```

but is represented by these runes:

```
[72 101 108 108 111 44 32 19990 30028]
```

as you can see if you range over it.

## Encodings

Most of the time when working with unicode strings in Go you won't have to do anything special. If you need to convert from another encoding like Windows 1252 you can use the [encoding](https://godoc.org/golang.org/x/text/encoding) package under golang.org/x/text. There are other utility packages for dealing with text there.

## Strings are immutable

Strings in Go are immutable, you are not allowed to change the individual bytes. You can copy the string

## Strings are never nil

They have a zero value of "", and you cannot assign nil to a string. 

