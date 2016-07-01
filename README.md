# vim-go-tutorial

Tutorial for vim-go. A simple tutorial on how to install and use vim-go.

# Quick Setup

We're going to use `vim-plug` to install vim-go. Feel free to use other plugin
managers as well. The vimrc is very minimalistic, but we'll going to improve
with time as you proceed with the tutorial. But 

First fetch and install `vim-plug` along with `vim-go`:

```
curl -fLo ~/.vim/autoload/plug.vim --create-dirs https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim
git clone https://github.com/fatih/vim-go.git ~/.vim/plugged/vim-go
```

Create `~/.vimrc` with following content:

```
call plug#begin()
Plug 'fatih/vim-go'
call plug#end()
```

And finally be sure to install all necessary Go tools (such as guru, goimports,
gocode, etc...). If you already have them in your PATH, you're good to go.

```
vim -c "GoInstallBinaries" -c "qa" 
```

or open vim and execute `:GoInstallBinaries`

For the tutorial, all our examples will be under
`GOPATH/src/github.com/fatih/vim-go-tutorial/`. Please be sure your inside this
folder, if not create the folder and clone the tutorial there. This will make
it easy to follow the tutorial.

If you already have a `GOPATH` setup just execute:

```
go get github.com/fatih/vim-go-tutorial
```

# Hello World!

Open a new file from your Terminal:

```
vim main.go
```

As you see vim-go automatically populated the content with a simple main
package. Save the file with `:w`.

# Run it

You can easily run the file with `:GoRun`. Under the hood it calls `go run` for
the current file. You should see that it prints `vim-go`.

# Build it

Replace `vim-go` with `Hello Gophercon`. Let us compile the file instead of running it.
For this we have `:GoBuild`. If you call it, you should see a message in form:

```
vim-go: [build] SUCCESS
```

Under the hood it calls `go build`, but it's more smarter. It does a couple of
things different: 

* No binaries are created, you can call `:GoBuild` multiple times without
  polluting your workspace.
* It automatically cd's into the source package's directory
* It parses any errors and shows them inside a quickfix list 
* It automatically detects the GOPATH and modifies it if needed (detects
  projects such as `gb`, `Godeps`, etc.. )
* Uses [vim-dispatch](https://github.com/tpope/vim-dispatch) if installed.
* Runs async if used within NeoVim (coming soon to Vim as well!)

# Fix it

Let's introduce two errors by adding two compile errors:

```
var b = foo()

func main() {
	fmt.Println("Hello GopherCon")
	a
}
```

save the file and call `:GoBuild` again. 

This time the quickfix view will be opened. To jump between the errors you can
use `:cnext` and `:cprevious`. Let us remove those the first error, save the
file and call `:GoBuild` again. You'll see the quickfix list is updated with a
single error. Remove the second error as well, save the file and call
`:GoBuild` again. Now because there are no more errors, vim-go automatically
closes the quickfix window for you.

Let us improve it a little bit. Vim has a setting called `autowrite` that
writes the content of the file automatically if you call `:make`. vim-go also
makes use of this setting. Open your `.vimrc` and add the following:

```
set autowrite
```

Now you don't have to save your file anymore when you call `:GoBuild`.  If we
introduce back the two errors and call `:GoBuild`, we can now iterate much more
faster by only calling `:GoBuild`.

`:GoBuild` jumps to the first error encountered. If you don't want to jump
append the `!` (bang) sign: `:GoBuild!`.

In all the `go` commands, such as `:GoRun`, `:GoInstall`, `:GoTest`, etc..,
whenever there is an error the quickfix window always will pop up.

## .vimrc improvements:

You can add some shorcuts to make it easier to jump between errors in quickfix
list:

```
map <C-n> :cn<CR>
map <C-m> :cp<CR>
nnoremap <leader>a :cclose<CR>
```

I also use these shorcuts to build and run a Go program with `<leader>b` and
`<leader>r`:

```
autocmd FileType go nmap <leader>b  <Plug>(go-build)
autocmd FileType go nmap <leader>r  <Plug>(go-run)
```

# Test it

Let's write a simple function and a test that tests the function. Add the following:


```
func Bar() string {
	return "bar"
}
```

open a new file called `main_test.go` (it doesn't matter how you open it, from
inside vim, a separate Vim session, etc.. it's up to you). Let us use the
curent buffer and open it from vim via `:edit main_test.go`.

When you open the new file you notice something. The file automatically has the
package declaration added:

```
package main
```

This is done by vim-go automatically. This time it didn't create a sample
application, instead it detected that the file is inside a valid package and
therefore it created a file based on the package name (in our case the package
name was `main`)

Complement the test file with to the following code to:

```
package main

import (
	"testing"
)

func TestBar(t *testing.T) {
	result := Bar()
	if result != "bar" {
		t.Errorf("expecting bar, got %s", result)
	}
}
```

And call `:GoTest`. You'll see the following message:

```
vim-go: [test] PASS
```

`:GoTest` calls `go test` under the hood. It has the same improvements just
like we have for `:GoBuild`. If there is any test error, a quickfix list is
opened again and you can jump to it easily.


# Edit it (DRAFT)

You'll see we get an error because the `testing` package is not imported
(intentionally). We can easily go and edit the file, but instead we're going to
use the vim command `:GoImport`. This command adds the given package to the
import path. Run it via: `:GoImport testing`. You'll see the `testing` package
is being added. We also have `:GoImportAs` and `:GoDrop` to edit the import
paths. 



## Commands

# Go Cmd
* :GoBuild
* :GoRun
* :GoInstall
* :GoGenerate
* :GoTest
* :GoTestFunc
* :GoTestCompile
* :GoCoverage
* :GoCoverageToggle
* :GoCoverageClear
* :GoCoverageBrowser

# Editing
* :GoImport
* :GoImportAs
* :GoDrop
* :GoFmt
* :GoImports
* :GoRename
* :GoImpl

# Lint
* :GoLint
* :GoVet
* :GoErrCheck
* :GoMetaLinter

# Documentation/Exploring
* :GoDoc
* :GoDocBrowser
* :GoAlternate
* :GoDecls
* :GoDeclsDir
* :GoDef
* :GoDefPop
* :GoDefStack
* :GoDefStackClear

* :GoInfo
* :GoCallees
* :GoCallers
* :GoDescribe
* :GoCallstack
* :GoFreevars
* :GoChannelPeers
* :GoReferrers
* :GoGuruScope
* :GoGuruTags

# Others
* :GoPath
* :GoFiles
* :GoDeps
* :GoPlay (accepts range)

* :GoInstallBinaries
* :GoUpdateBinaries
* :AsmFmt
