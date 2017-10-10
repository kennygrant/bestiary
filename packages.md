# Packages

In Go packages are a way of scoping code, and map exactly to folders. Within a package, code can refer to any identifier within that package, public or private, while those importing the package may only reference public identifiers beginning with an uppercase letter. Each package can contain multiple Go files, and each Go file in a package begins with a declaration at the top of the file for the form:

```
package x
```

where x is the package name. The name should be short, explicit and avoid punctuation - see [Package Names](https://blog.golang.org/package-names) on the Go blog.

## Packages vs Dirs

Packages are a directory on the file system containing .go files. **You cannot have multiple packages within one directory**. By convention, the package name is the same as the directory name, and the directory should contain a file of the package name with the .go suffix which contains the primary exports. You can if necessary rename packages on import and use a different package name to the folder name.

## Blank imports

You can import packages with the blank identifier solely for their init function, to do this prefix the import with \_

```go
import _ "net/http/pprof"
```

This makes it clear the package is unused but the init function will be called \(which may introduce side effects in this case registering handlers\).

## Don't import main

You should never try to import the main package. Always try to import from the top level down, so your lowest level packages know nothing about the levels above. This is not always possible but is a good guideline to limit fragile web of dependencies.

## Cyclic dependencies

If you run into cyclic imports, you have a package A which imports package B which then imports package A. To avoid this, try to keep your imports in one direction, importing small packages into the top level one. For example for a git command line tool you might have a main command, which imported a package which contains all the git specific structures and could be used as a library by that command, a server or any other type of app. The git package should never know about packages which import it.

If you structure your apps this way you will avoid cyclic imports and you should consider them a hint that something is wrong with your design.

## Using goimports

If you choose to use [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) instead of go fmt on save, and have several packages with the same name at different places in your `GOPATH`, it may not choose the correct imports, or even those closest to the importing package. There is a [fix](https://github.com/golang/go/issues/17557) almost ready for this, but that leaves the problem of imports from third party packages which might be replaced by an import from another package with the same name and similar code by mistake. So if you use this tool use it with caution and always check the imports inserted - it chooses the shortest name.

## Using go get

You should be aware that go get fetches the most recent version of any go package. This means if you are sharing code and want reproducible builds, you need to use the vendor directory. In practice because of the Go culture of no breaking changes this is rarely a problem, but sometimes APIs change, and this can lead to subtle breakage in your program if you are not careful. Try to vendor any dependencies you don't control to avoid this problem.

## Vendoring packages

There is no official package manager tool as yet in Go, though work is progressing on one \(go [dep](https://github.com/golang/dep)\). For now you can vendor your packages inside a vendor directory in your project, and those imports will be used instead of the imports currently. You can read more about this behaviour in the go command documentation under [Vendor Directories](https://golang.org/cmd/go/#hdr-Vendor_Directories).

## Internal directories

A subdirectory named 'internal' can be used for packages that can only be imported by nearby code, not by any other package. Packages in an internal directory are only accessible to those in the same file tree, rooted at the parent of the internal directory. So if one project includes 'myproj/example/internal/foo' only packages under myproj can import foo. 

This is typically used for dependencies which should not be shared and are for internal use only \(for example libraries with an unstable api\). 

## Avoid importing unsafe or reflect

If you can, avoid using the unsafe or reflect packages in your code – this also applies to libraries you import. 

* Package unsafe bypasses the Go type system. Packages that import unsafe may be non-portable and are not protected by the Go 1 compatibility guidelines. 
* Package reflect also bypasses the Go type system, so programs using it don't have the benefit of strict type checks, programs that use it will typically be slower and more fragile, and may use interface{} too much. It can be temping when writing a library to use reflect and attempt to handle several types in the same function \(for example an array of types or a single type\), but this makes it confusing for users \(no indication of what type should be passed in\), and easy to miss a type and cause panics. 

## func init

Packages can contain \(one or several\) init\(\) functions to set up state, these are run after all the imports and package variables have been initialised:

```go
// init() loads the configuration for this package
func init() {
 if user == "" {
        log.Fatal("$USER not set")
 }
 if home == "" {
    home = "/home/" + user
 }
}
```

Avoid using this if you can, and particularly avoid using it to set global state \(like set flags in other packages, or attach handlers to the http.DefaultServeMux\) - doing so can lead to subtle bugs. The init function should be solely concerned with the package it is in.

## Flags

Libraries and packages should not define their own flags or use the flags package. Instead take parameters to define behaviour - the application \(main\) should be in charge of parsing flags and passing the chosen configuration values in to a package. Otherwise it will be difficult to reuse the package or import it anywhere else.



