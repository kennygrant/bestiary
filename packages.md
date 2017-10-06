# Packages

In go packages are a way of scoping code, and map exactly to folders. Within a package, code can refer to any identifier within that package, public or private, while those importing the package may only reference public identifiers beginning with an uppercase letter. Each package can contain multiple go files, and each go file in a package begins with a declaration at the top of the file for the form:

```
package x
```

where x is the package name. The name should be short, explicit and avoid punctuation - see [Package Names](https://blog.golang.org/package-names) on the Go blog.

## Packages vs Dirs

Packages are a directory on the file system containing .go files. You cannot have multiple packages within one directory. By convention, the package name is the same as the directory name, and the directory should contain a file of the package name with the .go suffix which contains the primary exports. You can if necessary rename packages on import.

## Don't import main

You should never try to import the main package. Always try to import from the top level down, so your lowest level packages know nothing about the levels above. This is not always possible but is a good guideline to limit fragile web of dependencies.

## Cyclic dependencies

If you run into cyclic imports, you have a package A which imports package B which then imports package A. To avoid this, try to keep your imports in one direction, importing small packages into the top level one. For example for a git command line tool you might have a main command, which imported a package which contains all the git specific structures and could be used as a library by that command, a server or any other type of app.

If you structure your apps this way you will avoid cyclic imports and you should consider them a hint that something is wrong with your design.

## Using goimports

If you choose to use [goimports](https://godoc.org/golang.org/x/tools/cmd/goimports) instead of go fmt on save, and have several packages with the same name at different places in your `GOPATH`, it may not choose the correct imports, or even those closest to the importing package. There is a [fix](https://github.com/golang/go/issues/17557) almost ready for this, but that leaves the problem of imports from third party packages which might be replaced by an import from another package with the same name and similar code by mistake. So if you use this tool use it with caution and always check the imports inserted.

## Using go get

You should be aware that go get fetches the most recent version of any go package. This means if you are sharing code and want reproducible builds, you need to use the vendor directory. In practice because of the Go culture of no breaking changes this is rarely a problem, but sometimes APIs change, and this can lead to subtle breakage in your program if you are not careful. Try to vendor any dependencies you don't control.

## Vendoring packages

There is no official package manager tool as yet in Go, though work is progressing on one \(go [dep](https://github.com/golang/dep)\). For now you can vendor your packages inside a vendor directory in your project, and those imports will be used instead of the imports currently. You can read more about this behaviour in the go command documentation under [Vendor Directories](https://golang.org/cmd/go/#hdr-Vendor_Directories).

