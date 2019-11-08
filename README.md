# thinkvim-study

`ThinkVim`是一个很好的`Vim`配置文件项目，用来拆分`vim`的配置。

原作者github项目:<https://github.com/hardcoreplayers/ThinkVim> 

这个工程来记录学习这个项目笔记。

自己由于不懂`Vim Script`这个也是记录学习`Vim Script`的笔记。

## 1、`init.vim`

这个文件是`neovim`的配置文件，是整个项目的入口

`init.vim`

```ruby
execute 'source' fnamemodify(expand('<sfile>'), ':h').'/core/vimrc'
```

- [execute](./note/execute.md):执行字符串命令

- [fnamemodify](./note/fnamemodify.md):获取路径内置函数
- [expand](./note/expand.md):获取路径内置函数

所以这里是加载`./core/vimrc`配置文件

## 2、`core/vimrc`

```bash
if &compatible
	" vint: -ProhibitSetNoCompatible
	set nocompatible
	" vint: +ProhibitSetNoCompatible
endif

" Set main configuration directory as parent directory
let $VIM_PATH = fnamemodify(resolve(expand('<sfile>:p')), ':h:h')
let s:user_settings_path = expand('~/.thinkvim.d/local_settings.vim')

" Regular Vim doesn't add custom configuration directories, if you use one
if &runtimepath !~# $VIM_PATH
	set runtimepath^=$VIM_PATH
endif

let $DATA_PATH = g:etc#cache_path

" Set augroup
augroup MyAutoCmd
	autocmd!
augroup END

" Disable vim distribution plugins
let g:loaded_getscript = 1
let g:loaded_getscriptPlugin = 1
let g:loaded_gzip = 1
let g:loaded_logiPat = 1
let g:loaded_matchit = 1
let g:loaded_matchparen = 1
let g:netrw_nogx = 1 " disable netrw's gx mapping.
let g:loaded_rrhelper = 1
let g:loaded_shada_plugin = 1
let g:loaded_tar = 1
let g:loaded_tarPlugin = 1
let g:loaded_tutor_mode_plugin = 1
let g:loaded_2html_plugin = 1
let g:loaded_vimball = 1
let g:loaded_vimballPlugin = 1
let g:loaded_zip = 1
let g:loaded_zipPlugin = 1

" Initialize base requirements
if has('vim_starting')
	" Global Mappings "{{{
	" Use spacebar as leader and ; as secondary-leader
	" Required before loading plugins!
	let g:mapleader="\<Space>"
	let g:maplocalleader=';'

	" Release keymappings prefixes, evict entirely for use of plug-ins.
	nnoremap <Space>  <Nop>
	xnoremap <Space>  <Nop>
	nnoremap ,        <Nop>
	xnoremap ,        <Nop>
	nnoremap ;        <Nop>
	xnoremap ;        <Nop>
	nnoremap m        <Nop>
	xnoremap m        <Nop>
   if ! has('nvim') && has('pythonx')
		if has('python3')
			set pyxversion=3
		elseif has('python')
			set pyxversion=2
		endif
	endif


	" Ensure data directories
	call etc#util#ensure_directory([
		\   g:etc#cache_path . '/undo',
		\   g:etc#cache_path . '/backup',
		\   g:etc#cache_path . '/session',
		\   g:etc#vim_path . '/spell'
		\ ])

endif

call etc#init()
call etc#util#source_file('core/plugins/allkey.vim')
call etc#util#source_file('core/general.vim')
call etc#util#source_file('core/filetype.vim')
call etc#util#source_file('core/mappings.vim')

" Initialize user favorite colorscheme
call theme#init()
call etc#util#source_file('core/color.vim')

function! s:check_custom_settings(filename)abort
       let  content = readfile(a:filename)
       if empty(content)
           return 0
       endif
       return 1
endfunction

function! s:source_custom(path, ...) abort
  let use_global = get(a:000, 0, !has('vim_starting'))
  let abspath = resolve(expand('~/.thinkvim.d' . a:path))
  if !use_global
    execute 'source' fnameescape(abspath)
    return
  endif

  " substitute all 'set' to 'setglobal'
  let content = map(readfile(abspath),
        \ 'substitute(v:val, "^\\W*\\zsset\\ze\\W", "setglobal", "")')
  " create tempfile and source the tempfile
  let tempfile = tempname()
  try
    call writefile(content, tempfile)
    execute 'source' fnameescape(tempfile)
  finally
    if filereadable(tempfile)
      call delete(tempfile)
    endif
  endtry
endfunction
" Load user custom local settings
if filereadable(s:user_settings_path)
	if s:check_custom_settings(s:user_settings_path)
		call s:source_custom('/local_settings.vim')
	endif
endif


set secure

" vim: set ts=2 sw=2 tw=80 noet :

```

- 不与`vi`兼容

  ```bash
  if &compatible
  	" vint: -ProhibitSetNoCompatible
  	set nocompatible
  	" vint: +ProhibitSetNoCompatible
  endif
  ```

- 设置配置文件目录

  ```shell
  " Set main configuration directory as parent directory
  let $VIM_PATH = fnamemodify(resolve(expand('<sfile>:p')), ':h:h')
  let s:user_settings_path = expand('~/.thinkvim.d/local_settings.vim')
  ```

- 设置runtime路径

  参考：<http://blog.chinaunix.net/uid-22695386-id-2378701.html> 

  参考：[runtime](./note/runtimepath.md)

  ```shell
  " Regular Vim doesn't add custom configuration directories, if you use one
  if &runtimepath !~# $VIM_PATH
  	set runtimepath^=$VIM_PATH
  endif
  ```

- 设置数据目录

  ```shell
  let $DATA_PATH = g:etc#cache_path
  ```

  不知道这个用来做什么

  关于`#`参考[自动加载](./note/autoload.md)

- [创建自动命令组](./note/augroup.md)

  清楚命令组MyAutoCmd

  ```shell
  " Set augroup
  augroup MyAutoCmd
  	autocmd!
  augroup END
  ```

  对于这个问题，Vim有一个解决方案。这个解决方案的第一步是将相关的自动命令收集起来放到一个已命名的组（groups）中。

  新开一个Vim实例，这样可以清除之前所创建的自动命令。

- 禁用分发插件

  ```shell
  " Disable vim distribution plugins
  let g:loaded_getscript = 1
  let g:loaded_getscriptPlugin = 1
  let g:loaded_gzip = 1
  let g:loaded_logiPat = 1
  let g:loaded_matchit = 1
  let g:loaded_matchparen = 1
  let g:netrw_nogx = 1 " disable netrw's gx mapping.
  let g:loaded_rrhelper = 1
  let g:loaded_shada_plugin = 1
  let g:loaded_tar = 1
  let g:loaded_tarPlugin = 1
  let g:loaded_tutor_mode_plugin = 1
  let g:loaded_2html_plugin = 1
  let g:loaded_vimball = 1
  let g:loaded_vimballPlugin = 1
  let g:loaded_zip = 1
  let g:loaded_zipPlugin = 1
  ```

  从注释上来下，是做这个的，以后回头再来看这些变量的作用

- 开始时初始化设置

  ```css
  " Initialize base requirements
  if has('vim_starting')
  	" Global Mappings "{{{
  	" Use spacebar as leader and ; as secondary-leader
  	" Required before loading plugins!
  	let g:mapleader="\<Space>"
  	let g:maplocalleader=';'
  
  	" Release keymappings prefixes, evict entirely for use of plug-ins.
  	nnoremap <Space>  <Nop>
  	xnoremap <Space>  <Nop>
  	nnoremap ,        <Nop>
  	xnoremap ,        <Nop>
  	nnoremap ;        <Nop>
  	xnoremap ;        <Nop>
  	nnoremap m        <Nop>
  	xnoremap m        <Nop>
     if ! has('nvim') && has('pythonx')
  		if has('python3')
  			set pyxversion=3
  		elseif has('python')
  			set pyxversion=2
  		endif
  	endif
  
  
  	" Ensure data directories
  	call etc#util#ensure_directory([
  		\   g:etc#cache_path . '/undo',
  		\   g:etc#cache_path . '/backup',
  		\   g:etc#cache_path . '/session',
  		\   g:etc#vim_path . '/spell'
  		\ ])
  
  endif
  ```

  设置`leader`

  参考：[Leader](./note/leaders.md)

  ```css
  let g:mapleader="\<Space>"
  let g:maplocalleader=';'
  ```

  











# 未知的设置

## 1、get()

```css
let g:etc#vim_path =
	\ get(g:, 'etc#vimpath',
	\   exists('*stdpath') ? stdpath('config') :
	\   ! empty($MYVIMRC) ? fnamemodify(expand($MYVIMRC), ':h') :
	\   ! empty($VIMCONFIG) ? expand($VIMCONFIG) :
	\   ! empty($VIMCONFIG) ? expand($VIMCONFIG) :
	\   ! empty($VIM_PATH) ? expand($VIM_PATH) :
	\   expand('$HOME/.vim')
	\ )
```

# 一些问题

## 1、BufOnly映射

mapping.vim

```css
"buffer
nnoremap <leader>bc :BufOnly<CR>
nnoremap <Leader>bo :BufOnly 
```

这个映射是如何生效的？

## 2、EditPluginSetting

mapping.vim

```css
" a command which  edit PLugin config easy
nnoremap <leader>p :EditPluginSetting <Space>
```

这个映射如何使用？

## 3、buffer select mapping

mapping.vim

```css
nmap <leader>1 <Plug>BuffetSwitch(1)
nmap <leader>2 <Plug>BuffetSwitch(2)
nmap <leader>3 <Plug>BuffetSwitch(3)
nmap <leader>4 <Plug>BuffetSwitch(4)
nmap <leader>5 <Plug>BuffetSwitch(5)
nmap <leader>6 <Plug>BuffetSwitch(6)
nmap <leader>7 <Plug>BuffetSwitch(7)
nmap <leader>8 <Plug>BuffetSwitch(8)
nmap <leader>9 <Plug>BuffetSwitch(9)
nmap <leader>0 <Plug>BuffetSwitch(10)
```

这个映射如何生效？



## 参考

- <https://v2ex.com/t/458103> 
- <http://www.treelib.com/book-detail-id-47-aid-2868.html> 
- <https://www.v2ex.com/t/537934> 
- <http://www.ruanyifeng.com/blog/2018/09/vimrc.html> 