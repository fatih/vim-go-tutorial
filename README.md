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


Open a new file:

```
vim main.go
```

and voila! vim-go automatically 


## Commands

# Editing
:GoImport
:GoImportAs
:GoDrop
:GoFmt
:GoImports
:GoRename
:GoImpl

# Go Cmd
:GoBuild
:GoRun
:GoInstall
:GoGenerate
:GoTest
:GoTestFunc
:GoTestCompile
:GoCoverage
:GoCoverageToggle
:GoCoverageClear
:GoCoverageBrowser

# Lint
:GoLint
:GoVet
:GoErrCheck
:GoMetaLinter

# Documentation/Exploring
:GoDoc
:GoDocBrowser
:GoAlternate
:GoDecls
:GoDeclsDir
:GoDef
:GoDefPop
:GoDefStack
:GoDefStackClear

:GoInfo
:GoCallees
:GoCallers
:GoDescribe
:GoCallstack
:GoFreevars
:GoChannelPeers
:GoReferrers
:GoGuruScope
:GoGuruTags

# Others
:GoPath
:GoFiles
:GoDeps
:GoPlay (accepts range)

:GoInstallBinaries
:GoUpdateBinaries
:AsmFmt
