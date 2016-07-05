# vim-go-tutorial

Tutorial for vim-go. A simple tutorial on how to install and use vim-go.

# Table of Contents

1. [Quick Setup](#quick-setup)
2. [Hello World](#hello-world)
3. [Run it](#run-it)
4. [Build it](#build-it)
5. [Fix it](#fix-it)
6. [Test it](#test-it)
7. [Cover it](#cover-it)
8. [Edit it](#edit-it)
  * [Imports](#imports)
  * [Text Objects](#text-objects)
  * [Struct split&join](#struct-split-and-join)
  * [Snippets](#snippets)
9. [Beautify it](#beautify-it)
10. [Check it](#check-it)
11. [Navigate it](#Navigate-it)
12. [Understand it](#Understand-it)

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

```vim
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

```go
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

### vimrc improvements

You can add some shorcuts to make it easier to jump between errors in quickfix
list:

```vim
map <C-n> :cn<CR>
map <C-m> :cp<CR>
nnoremap <leader>a :cclose<CR>
```

I also use these shorcuts to build and run a Go program with `<leader>b` and
`<leader>r`:

```vim
autocmd FileType go nmap <leader>b  <Plug>(go-build)
autocmd FileType go nmap <leader>r  <Plug>(go-run)
```

# Test it

Let's write a simple function and a test that tests the function. Add the following:


```go
func Bar() string {
	return "bar"
}
```

open a new file called `main_test.go` (it doesn't matter how you open it, from
inside vim, a separate Vim session, etc.. it's up to you). Let us use the
curent buffer and open it from vim via `:edit main_test.go`.

When you open the new file you notice something. The file automatically has the
package declaration added:

```go
package main
```

This is done by vim-go automatically. This time it didn't create a sample
application, instead it detected that the file is inside a valid package and
therefore it created a file based on the package name (in our case the package
name was `main`)

Complement the test file with to the following code to:

```go
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

Another small improvement is that you don't have to open the test file itself.
Try it yourself, open `main.go` and call `:GoTest`. You'll see the tests will
be run for you as well.

`:GoTest` by default times out after 10 seconds. This is useful because Vim is
not async by default. You can change the timeout value with `let g:go_test_timeout = 10s`

We have two more commands that makes it easy to deal with test files. The other
one is `:GoTestFunc`. This only tests the function immediate to your cursor.
Let us change the content of the test file (`main_test.go`) to:

```go
package main

import (
	"testing"
)

func TestFoo(t *testing.T) {
	t.Error("intentional error 1")
}

func TestBar(t *testing.T) {
	result := Bar()
	if result != "bar" {
		t.Errorf("expecting bar, got %s", result)
	}
}

func TestQuz(t *testing.T) {
	t.Error("intentional error 2")
}
```

Now when we call `:GoTest` a quickfix window wil be open with two errors.
However if go inside the `TestBar` function and call `:GoTestFunc`, you'll see
that our test passes!  This is really useful if you have a lot of tests that
takes time and you only want to test certain tests.

The other test related command is `:GoTestCompile`. Tests not only needs to
pass with success, it also needs to compile without any problems.
`:GoTestCompile` compiles your test file, just like `:GoBuild` and opens a
quickfix if there is any errors. This however **doesn't run** the tests. This
is very useful if you have a large test which you're editing a lot. Call
`:GoTestCompile` in the current test file, you should see the following:

```
vim-go: [test] PASS
```

### vimrc improvements

* As with `:GoBuild` we can add a mapping to easily call `:GoTest` with a key
combination. Add the following to your `.vimrc`:

```vim
autocmd FileType go nmap <leader>t  <Plug>(go-test)
```

Now you can easily test your files via `<leader>t`

* Let's make building Go files simpler. First, remove the following mapping we added
  previously:

```vim
autocmd FileType go nmap <leader>b  <Plug>(go-build)
```

We're going to add a mapping that is better improved. To make it seamless for
any Go file we can create a simple Vim function that checks the type of the Go
file, and execute `:GoBuild` or `:GoTestCompile`.  Below is the helper function
you can add to your `.vimrc` which does it:

```vim
" run :GoBuild or :GoTestCompile based on the go file
function! s:build_go_files()
  let l:file = expand('%')
  if l:file =~# '^\f\+\.go$'
    call go#cmd#Build(0)
  elseif l:file =~# '^\f\+_test\.go$'
    call go#cmd#Test(0, 1)
  endif
endfunction

autocmd FileType go nmap <leader>b :<C-u>call <SID>build_go_files()<CR>
```

Now whenever you hit `<leader>b` it'll build either your Go file or it'll
compile your test files seamlessly.

* By default the leader shortcut is defined as: `\` I've mapped my leader to
`,` as I find it more useful with the following setting(put this in the
beginning of .vimrc):

```vim
let mapleader = ","
```

So with this setting, we can easily build any test and non test files with `,b`.

# Cover it

Let's dive in even more in the world of tests. Tests are really important. Go
has a really great way of showing the coverage of your source code. vim-go
makes it easy to see the code coverage without leaving Vim in a very elegant
way.


Let's first change back our `main_test.go` file to:

```go
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

And `main.go` to


```go
package main

func Bar() string {
	return "bar"
}

func Foo() string {
	return "foo"
}

func Qux(v string) string {
	if v == "foo" {
		return Foo()
	}

	if v == "bar" {
		return Bar()
	}

	return "INVALID"
}
```

Now let us call `:GoCoverage`. Under the hood this calls `go test -coverprofile
tempfile`. It parses the lines from the profile and then dynamically changes
the syntax of your source code to reflect the coverage. As you see, because we
only have a test for the `Bar()` function, that is the only function it's
green. 

To clear the syntax highlighting you can call `:GoCoverageClear`. Let us add a
test case and see how the coverage changes. Add the following to `main_test.go`
to:

```go
func TestQuz(t *testing.T) {
	result := Qux("bar")
	if result != "bar" {
		t.Errorf("expecting bar, got %s", result)
	}

	result = Qux("qux")
	if result != "INVALID" {
		t.Errorf("expecting INVALID, got %s", result)
	}
}
```

If we call `:GoCoverage` again, you'll see that the `Quz` function is now
tested as well and that it has a larger coverage. Call `:GoCoverageClear` again
to clear the syntax highlighting.

Because calling `:GoCoverage` and `:GoCoverageClear` are used a lot together,
there is another command that makes it easy to call and clear the result. You
can also use `:GoCoverageToggle`. This acts as a toggle and shows the coverage,
and when called again it clears the coverage.  It's up to your workflow how you
want to use them.

Finally, if you don't like vim-go's internal view, you can also call
`:GoCoverageBrowser`. Under the hood it uses `go tool cover` to create a HTML
page and then opens it in your default browser. Some people like this more.

Using the `:GoCoverageXXX` commands do not create any kind of temporary files.
It doesn't pollute your workflow. So you don't have to deal with removing
unwanted files everytime.

### vimrc improvements

Add the following to your `.vimrc`:

```vim
autocmd FileType go nmap <Leader>c <Plug>(go-coverage-toggle)
```

With this you can easily call `:GoCoverageToggle` with `<leader>c`


# Edit it

### Imports

Let us start with a sample `main.go` file:

```go
package main

     import "fmt"

func main() {
 fmt.Println("    gophercon"     )
}
```

Let's start with something we know already. If save the file, you'll see that
it'll be formatted automatically. It's enabled by default but can be disables
if wished (not sure why you would though :)) with `let g:go_fmt_autosave = 0`.
Optionally we also provide `:GoFmt` command, which goes and runs `:GoFmt` under
the hood.

Let's print the `"gophercon"` string all uppercase. For it we're going to use
the `strings` package. Change the definition to:

```go
fmt.Println(strings.ToLower("Gopher"))
```

When you build it you'll get an error of course:

```
main.go|8| undefined: strings in strings.ToLower
```

You'l see we get an error because the `strings` package is not imported. Vim-go
has couple of commands to make it easy to manipulate the import declarations.

We can easily go and edit the file, but instead we're going to use the vim
command `:GoImport`. This command adds the given package to the import path.
Run it via: `:GoImport strings`. You'll see the `strings` package is being
added.  The greatin thing about this command is that it also supports
completion. So you can just type `:GoImport s` and hit tab.

We also have `:GoImportAs` and `:GoDrop` to edit the import paths.
`:GoImportAs` is the same as `:GoImport`, but it allowd to change the
packagename. For example `:GoImportAs str strings`, will import `strings` with
the package name `str.`

Finally `:GoDrop` makes it easy to remove any import paths from the import
declarations. `:GoDrop strings` will remove it from the import declarations.

Of course manipulating import paths is so 2010. We have better tools to handle
the case for us. If you didn't heard it yet, it's called `goimports`.
`goimports` is a replacement for `gofmt`. You have two ways of using it. The
first way (also recommended way) is telling vim-go to use it when saving the
file:

```
let g:go_fmt_command = "goimports"
```

Now whenever you save your file, `goimports` will automatically format and also
rewrite your import declarations. Some people do not prefer `goimports` as it
might be slow on very large code bases. In this case we also have the
`:GoImports` command (note the `s` at the end). With this, you can explicitly
call `goimports`


### Text objects

Let us show more editing tips/tricks. There are two text objects that we can
use to change functions. Those are `if` and `af`. `if` means inner function and
it allows you to select the content of a function enclosure. Change your `main.go` file to:


```go
package main

import "fmt"

func main() {
	fmt.Println(1)
	fmt.Println(2)
	fmt.Println(3)
	fmt.Println(4)
	fmt.Println(5)
}
```

Put your cursor on the `func` keyword  Now execute the following in `normal`
mode and see what happens:

```
dif
```

You'll see that the function body is removed. Because we used the `d` operator.
Undo your changes with `u`. The great thing is that your cursor can be anywhere
starting from the `func` keyword until the right brace `}`. It uses the tool
[motion](https://github.com/fatih/motion) under the thood. A tool that I wrote
explicitly for vim-go to support features like this. It's Go AST aware and thus
it's capabilities are really good. Like what you might ask? Change `main.go` to:

```go
package main

import "fmt"

func Bar() string {
	fmt.Println("calling bar")

	foo := func() string {
		return "foo"
	}

	return foo()
}
```

Previously we were using regexp based text objects. And it lead to problems.
For example in this example, put your cursor to the anonymous functions' `func`
keyword and execute `dif` in `normal` mode. You'll see that only the body of
the anonymous function is deleted.

We only used so far the `d` operator (delete). However it's up to you. For
example you can select it via `vif` or yank(copy) with `yif`.

We also have `af`, which means `a function`. This text object includes the
whole function declaration. Change your `main.go` to:


```go
package main

import "fmt"

// bar returns a the string "foo" even though it's named as "bar". It's an
// example to be used with vim-go's tutorial to show the 'if' and 'af' text
// objects.
func bar() string {
	fmt.Println("calling bar")

	foo := func() string {
		return "foo"
	}

	return foo()
}
```


So here is the great thing. Because of `motion` we have full knowledge about
every single syntax node. Put your cursor on top of the `func`  keyword or
anywhere below or above (doesn't matter). If you now execute `vaf`, you'll see
that the function declaration is being selected, along with the doc comment as
well! You can for example delete the whole function with `daf`, and you'll see
that the comment is gone as well. Go ahead and put your cursor on top of the
comment and execute `vif` and then `vaf`. You'll see that it selects the
function body, even though your cursor is outside the function, or it selects
the function comments as well.

This is really powerful and this all is thanks to the knowledge we have from
let g:go_textobj_include_function_doc = 1 `motion`. If you don't like comments
being a part of the function declaration, you can easily disable it with:

```vim
let g:go_textobj_include_function_doc = 1
```

If you are interested more about `motion`, checkout the blog post I wrote with
more details: [Treating Go types as objects in Vim](https://medium.com/@farslan/treating-go-types-as-objects-in-vim-ed6b3fad9287#.45q2rtqgf)

(Optional question: without looking at the `go/ast` package, is the doc comment
a part of the function declaration or not?)

### Struct split and join
There is a great plugin that allows you to split or join Go structs. It's
actually not a Go plugin, but it has support for Go structs. To enable it add
plugin directove between the `plug` definition into your `vimrc` and run
`:PlugInstall`. Example:

```vim
call plug#begin()
Plug 'fatih/vim-go'
Plug 'AndrewRadev/splitjoin.vim'
call plug#end()
```

Once you have installed the plugin, change the `main.go` file too:

```go
package main

type Foo struct {
	Name    string
	Ports   []int
	Enabled bool
}

func main() {
	foo := Foo{Name: "gopher", Ports: []int{80, 443}, Enabled: true}
}
```

Put your cursor on the same line as the struct expression. Now type `gS`. This
will `split` the struct expression into multiple lines. And you can even
reverse it. If your cursor is still on the `foo` variable, execute `gJ` in
`normal` mode. You'll see that the field definitions are all joined.

This doesn't use any AST aware tools, so for example if you type `gJ` on top of
the fields, you'll see that only two files are joined.

### Snippets

Vim-go supports two popular snippet plugins.
[Ultisnips](https://github.com/SirVer/ultisnips) and
[neosnippet](https://github.com/Shougo/neosnippet.vim). By default out of the
box, if you have `Ultisnips` installed it'll work.  Let us install `ultisnips`
first. Add it between the `plug` directives in your `vimrc` and then run
`:PlugInstall`. Example:

```vim
call plug#begin()
Plug 'fatih/vim-go'
Plug 'SirVer/ultisnips'
call plug#end()
```

There are many helpful snippets. To see the full list check our current
snippets:
https://github.com/fatih/vim-go/blob/master/gosnippets/UltiSnips/go.snippets

Let me show some of the snippets that I'm using the most. Change your `main.go`
content to:

```go
package main

import "encoding/json"

type foo struct {
	Message    string
	Ports      []int
	ServerName string
}

func newFoo() (*foo, error) {
	return &foo{
		Message:  "foo",
		Ports: []int{80},
		ServerName: "Foo",
	}, nil
}

func main() {
	res, err := newFoo()

	out, err := json.Marshal(res)
}
```

Let's put our cursor just after the `newFoo()` expression. Let's panic here if
the err is non nil. Type `errp` in insert mode and just hit `tab`. You'll see
that it'll be expanded and put your cursor inside the `panic()`` function:

```
if err != nil {
    panic( )
          ^
          cursor position
}
```

Fill the panic with `err` and move on to the `json.Marshal` statement. Do the
same for it.

Now let us print the variable `out`. Because variable printing is so popular,
we have several snippets for it:

```
fn -> fmt.Println()
ff -> fmt.Printf()
ln -> log.Println()
lf -> log.Printf()
```

Here `ff` and `lf` are special. They dynamically copy the variable name into
the format string as well. Try it youself. Move your cursor to the end of the
main function and type `ff` and hit tab. After expanding the snippet you can
start typing. Type `string(out)` and you'll see that both the format string and
the variadic arguments will be filled with the same string you have typed.

This comes very handy to print quickly variables, especially for debugging.
Run your file with `:GoRun` and you should see the following output:

```
string(out) = {"Message":"foo loves bar","Ports":[80],"ServerName":"Foo"}
```

Great. Now let me show one last snippet that I think is very useful. As you see
from the output the fields `Message` and `Ports` are starting with an uppercase
character. To fix it we can add a json tag to the struct field. vim-go makes it
very easy to add field tags. Move your cursor to the end of the `Message`
string line in the field:

```
type foo struct {
    Message string .
                   ^ put your cursor here 
}
```

In `insert` mode, type `json` and hit enter. You'll see that it'll be
automatically expanded to valid field tag. And the field name is converted
automatically to a lowercase and put there for you. You should now see the
following:

```
type foo struct {
	Message  string `json:"message"`
}
```

It's really amazing. But we can be even more better! Go ahead and create a
snippet expansion for the `ServerName` field. You'll see that it's converted to
`server_name`. Quite amazing right! 

```go
type foo struct {
	Message    string `json:"message"`
	Ports      []int
	ServerName string `json:"server_name"`
}
```

You even change which case it should apply while converting. By default vim-go
uses `snake_case`. But you can also use `camelCase` if you wish. For example if
you wish to change the default value to camel case use the following setting:

```vim 
let g:go_snippet_case_type = "camelcase"
```

### vimrc improvements

* Don't forget to change `gofmt` to `goimports`

```vim
let g:go_fmt_command = "goimports"
```

* When you save your file, `gofmt` shows any errors during parsing the file. If
  there is any parse errors it'll show them inside a quickfix list. This is
  enabled by default. Some people don't like it. To disable it add:

```vim
let g:go_fmt_fail_silently = 1
```

# Beautify it

By default we only have a limited syntax highlighting enabled. There are two
main reasons. First is that people don't like much color. They prefer less
color as it causes to much distraction. The second reason is that it impacts
the performance of Vim a lot. We need to enable them explicitly. First add the
following settings to your `.vimrc`:


```
let g:go_highlight_types = 1
let g:go_highlight_fields = 1
let g:go_highlight_functions = 1
let g:go_highlight_methods = 1
```


The following setting will highlight operators, such as :

```
let g:go_highlight_operators = 1
```

```
let g:go_highlight_extra_types = 1
```


```
let g:go_highlight_format_strings = 1
let g:go_highlight_build_constraints = 1
let g:go_highlight_string_spellcheck = 1
let g:go_highlight_generate_tags = 0
```


### vimrc improvements

* Some people don't like how the tabs are shown. By default Vim shows `8`
  spaces for a single tab. However it's up to us how to represent in Vim. The
  following will change it to show a single tab as 4 spaces:

```
autocmd BufNewFile,BufRead *.go setlocal noexpandtab tabstop=4 shiftwidth=4 
```

This setting will not expand a tab into spaces. It'll show a single tab as `4`
spaces. It will use `4` spaces to represent a single indent.

* A lot of people ask for my colorscheme. I'm using a slightly modified
`molokai`. To enable it add the a Plug directive just between the plug
definitions:

```vim
call plug#begin()
Plug 'fatih/vim-go'
Plug 'fatih/molokai'
call plug#end()
```

Also add the following to enable molokai with original color scheme and 256
color version:

```vim
let g:rehash256 = 1
let g:molokai_original = 1
colorscheme molokai
```

After that restart your Vim and call `:PlugInstall`. This will pull the plugin
and install it for you. After the plugin is installed, you need to restart Vim
again.

# Check it

* :GoLint
* :GoVet
* :GoErrCheck
* :GoMetaLinter

# Navigate it

* :GoDoc
* :GoDocBrowser
* :GoAlternate
* :GoDef
* :GoDefPop
* :GoDefStack
* :GoDefStackClear
* :GoInfo

* :GoDecls
* :GoDeclsDir
ctrlp.vim
]]
[[

# Understand it


## Commands

# Editing
* :GoImport
* :GoImportAs
* :GoDrop
* :GoFmt
* :GoImports
* :GoRename
* :GoImpl

* af
* if

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

