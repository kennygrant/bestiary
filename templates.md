# Templates

The Go Standard Library offers templating of both text and html content. The html templates offer a superset of the text template features, including contextual escaping, but the interface can be a little confusing.  

## The missing docs

There is extensive documentation of the template packages, but most of it is in the [text/template](https://golang.org/pkg/text/template/) package, not the [html/template](https://golang.org/pkg/text/template/) package, so be sure to read that as well, even if you intend to focus on using html/template. 

## Delimeters

If you have problems with the go template delimeters clashing with other libraries, you can use the Delims function in order to set the delimiters to some other string. Note this must be done first, before files are parsed:

```go
tmpl, err := template.New("").Delims("[[", "]]").ParseFiles("foo.tmpl", "bar.tmpl")
```

## Escaping 

This package assumes that template authors are trusted, that Execute's data parameter is not, and seeks to preserve the properties below in the face of untrusted data:

If you need to bypass escaping, you can use the appropriate [typed string](https://golang.org/pkg/html/template/#hdr-Typed_Strings) like template.HTML in a function or field, to indicate to the package that this is trusted content. Be very careful not to use user content in such cases, or to escape it properly before use.  


## Indexing map and slice 



## Structs, fields and methods

There is a uniform interface for fields and methods of a struct - the . operator will call either a field or a method. The same is true of map entries. 

## Func Maps

Templates have function maps

Don't get too carried away adding functions to your templates though - they are deliberately bare bones so that most of the logic resides in your .go files, rather than in template files. 

## Prefix operators 

The boolean operators available in templates are all functions strictly speaking, this means they use prefix notation. Use and x y, not x and y. If you're coming from Ruby this may come as a surprise, as the function will work with one argument and may appear to work, but is subtly different in behaviour.  

```go
{{/* Don't do this, it will always be true if x is non nil */}}
{{ if x and y}}

{{ end }}

{{/* Do this instead to use and properly*/}}
{{ if and x y}}

{{ end }}
```

## Range functions 

In go templates you may range. Unlike the range in Go code. 

range with i extract

## Printing values 

The print and printf functions are available in templates. 

## Experimenting with templates 

If you'd like to experiment or try something out. You don't have to save separate files, you can define go templates in strings within your go code. 

If you have more than a few templates this can be cumbersome and you might want to consider storing them in files and readimng them into a template set. 

## The line eater

If you need to add newlines in your template for legibility (for example when outputing many columns of csv), you can invoke the line eater with the - symbol beside the braces (either start or end braces). Thus the template:

`  {{- .One }}
{{- .Two -}}
`

will output the values with whitespace removed:

> foobar

## Template Sets



## Nested Templates 

