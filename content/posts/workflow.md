---
layout: post
title: "My workflow with tmux and vim"
<!-- date: Mon, 01 Feb 2021 22:30:33 +0530 -->
categories: tmux vim
published: false
---

I usually do all my work is done inside a terminal.
Most of my work consists of either writing assignments for college or programming.

For prorgamming/writing I use [neovim][neovim] with tmux.  
[NeoVim][neovim] is a modern fork of Vim.  
The list of plugins I use is

```vim

call plug#begin()
" QOL
    Plug 'airblade/vim-rooter'
    Plug 'yuttie/comfortable-motion.vim'
    Plug 'tpope/vim-surround'
    Plug 'iamcco/markdown-preview.nvim', { 'do': { -> mkdp#util#install() }, 'for': ['markdown', 'vim-plug']}
" Fzf
    Plug 'junegunn/fzf'
    Plug 'junegunn/fzf.vim'
" Git plugin
    " Plug 'tpope/vim-fugitive'

" GUI Stuff
    Plug 'vim-airline/vim-airline'
    Plug 'vim-airline/vim-airline-themes'
	Plug 'tpope/vim-commentary'
    Plug 'uttarayan21/minimalist'  " Plug 'dikiaap/minimalist'
    Plug 'dracula/vim'
    Plug 'tpope/vim-vinegar'
    Plug 'mhinz/vim-crates'
" Intellisense
    Plug 'neoclide/coc.nvim', {'branch': 'release'}
" Ctags
    Plug 'ludovicchabant/vim-gutentags'
    Plug 'majutsushi/tagbar'
" Syntax Highlighting
    Plug 'rust-lang/rust.vim'
" Devicons
    Plug 'ryanoasis/vim-devicons'
" Text Objects
    Plug 'wellle/targets.vim'
" Dictionary
    Plug 'reedes/vim-wordy'
" Debug
    Plug 'epheien/termdbg'
call plug#end()
```

As for documents I write them in markdown and compile them to pdf using pandoc.

For emailing I use neomutt with gmail.
All my [dotfiles][dotfiles] for neovim,tmux,zsh and everything else is stored in my github [here][dotfiles]

[tmux]: https://github.com/tmux/tmux
[neovim]: https://github.com/neovim/neovim
[dotfiles]: https://github.com/uttarayan21/dotfiles
