# Packages

In Go packages are a way of scoping code, and map exactly to folders. Within a package, code can refer to any identifier within that package, public or private, while those importing the package may only reference public identifiers beginning with an uppercase letter. Each package can contain multiple Go files, and each Go file in a package begins with a declaration at the top of the file for the form:

```
// Package x ... this comment explains succinctly what the package does
package x
```

where x is the package name. The name should be short, explicit and avoid punctuation - see [Package Names](https://blog.golang.org/package-names) on the Go blog. The line preceding the package declaration can be used to document the package - typically this will be done in the most important file in the package, the one which has the same name as the package itself. 

## Packages vs Dirs

Packages are a directory on the file system containing .go files. **You cannot have multiple packages within one directory**. By convention, the package name is the same as the directory name, and the directory should contain a file of the package name with the .go suffix which contains the primary exports. You can if necessary rename packages on import and use a different package name to the folder name.

## Workspaces

A common cause of confusion for beginners with Go is the concept of workspaces, where all your go code lives. These are not checked into version control directly, and when beginning you will just have one workspace which is your gopath. This contains three dirs src, pkg, and bin, as detailed in the getting started section. Projects will typically be hosted under paths like this: src/host/username/project which are used as the import paths, and below that path can contain any directories you want to structure your project.

## Project structure

There are few restrictions on structure in a Go project, but for a beginner there are a few guidelines to keep in mind. The simplest structure is [one main package](https://golang.org/doc/code.html#Command) at the top level of your repository. This can be fetched by go get which is what users will expect. Beyond this you can structure it as you wish, with as many directories as you wish, though each directory with go files must be a separate package, and cannot contain more than one package.

A few guidelines to live by:

* Make your project go-gettable by having the main package at the top level
* Err on the side of fewer packages, not more
* Import lower packages from higher levels, and try to avoid interdependencies - low level packages should stand alone
* Store internal libraries not for export in a directoy named internal
* Vendor dependencies in a top level directory named vendor
* Always flatten vendor directories into this one top level folder

Packages can contain as many files as you want, not just one, and typically should be split by file. Don't split packages by type and have one type per package, a package will normall contain several types.

## Blank imports

You can import packages with the blank identifier solely for their init function, to do this prefix the import with \_

```go
import _ "net/http/pprof"
```

This makes it clear the package is unused but the init function will be called \(which may introduce side effects in this case registering handlers\).

## Don't import main

You should never try to import the main package. Always try to import from the top level down, so your lowest level packages know nothing about the levels above. This is not always possible but is a good guideline – avoid a fragile web of dependencies between internal packages which makes your program harder to reason about - prefer some copying to interdependent packages.

## Cyclic dependencies

If you run into cyclic imports, you have a package A which imports package B which then imports package A. To avoid this, try to keep your imports in one direction, importing small packages into the top level one. For example for a git command line tool you might have a main command, which imported a package which contains all the git specific structures and could be used as a library by that command, a server or any other type of app. The git package should never know about packages which import it.

If you structure your apps this way you will avoid cyclic imports and you should consider them a hint that something is wrong with your design.

## Using goimports

If you choose to use [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) instead of go fmt on save, and have several packages with the same name at different places in your `GOPATH`, it may not choose the correct imports, or even those closest to the importing package. 

There is a [fix](https://github.com/golang/go/issues/17557) almost ready for this, but that leaves the problem of imports from third party packages which might be replaced by an import from another package with the same name and similar code by mistake. So if you use the goimports tool use it with caution and always check the imports inserted - it chooses the shortest name by default and may import an unexpected package if you have homonym packages under gopath.

## Using go get

You should be aware that go get fetches the most recent version of any go package. This means if you are sharing code and want reproducible builds, you need to use the vendor directory. In practice because of the Go culture of no breaking changes this is rarely a problem, but sometimes APIs change, and this can lead to subtle breakage in your program if you are not careful. Try to vendor any dependencies you don't control to avoid this problem.

## Vendoring packages

There is no official package manager tool as yet in Go, though work is progressing on one \(go [dep](https://github.com/golang/dep)\). For now you can vendor your packages inside a vendor directory in your project, and those imports will be used instead of the imports currently. You can read more about this behaviour in the go command documentation under [Vendor Directories](https://golang.org/cmd/go/#hdr-Vendor_Directories).

## Internal directories

A subdirectory named 'internal' can be used for packages that can only be imported by nearby code, not by any other package. Packages in an internal directory are only accessible to those in the same file tree, rooted at the parent of the internal directory. So if one project includes 'myproj/example/internal/foo' only packages under myproj can import foo.

This is typically used for dependencies which should not be shared and are for internal use only \(for example libraries with an unstable api\).

## Don't import unsafe

If you can, avoid using the unsafe or reflect packages in your code – this also applies to libraries you import. Package unsafe bypasses the Go type system. Packages that import unsafe may be non-portable and are not protected by the Go 1 compatibility guidelines.

## Don't import reflect

Package reflect also bypasses the Go type system, so programs using it don't have the benefit of strict type checks, programs that use it will typically be slower and more fragile, and may use interface{} too much. It can be temping when writing a library to use reflect and attempt to handle several types in the same function \(for example an array of types or a single type\), but this makes it confusing for users \(no indication of what type should be passed in\), and easy to miss a type and cause panics.

```go
// Think twice!
import "reflect"

// Using reflect and empty interfaces
// obscures your intent for callers
// and subverts the type system
func MyAPIFunc(myvar interface{}) {
    // ... requires typecasts and perhaps reflect
}

```

## Cgo is not Go

Cgo is a useful crutch to allow calling into C libraries from Go and vice versa. Build times will be slower, and you won't be able to cross compile code easily. This throws away many of the advantages of writing code in Go. You can use it to do some very useful things like running Go programs on iOS or Android, or interfacing with large libraries of existing code written in C, and it is indispensable for those uses, but if you can avoid using it, do so.

## Syscall and cgo are not portable

If you're using cgo or syscall, your package probably isn't portable, and needs build tags for each platform you're going to target. Unlike other packages in the go ecosystem, the cgo or syscall packages are not portable across platforms. If you import them, you should be aware of this.

## Init funcs

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

