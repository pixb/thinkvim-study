# thinkvim-study

# 一、整体预览学习

`ThinkVim`是一个很好的`Vim`配置文件项目，用来拆分`vim`的配置。

原作者github项目:<https://github.com/hardcoreplayers/ThinkVim> 

这个工程来记录学习这个项目笔记。

自己由于不懂`Vim Script`这个也是记录学习`Vim Script`的笔记。

## 1、`init.vim`

这个文件是`neovim`的配置文件，是整个项目的入口

`init.vim`

```css
" 执行加载当前目录下core/vimrc配置文件
execute 'source' fnamemodify(expand('<sfile>'), ':h').'/core/vimrc'
```

- [execute](./note/execute.md):执行字符串命令

- [fnamemodify](./note/fnamemodify.md):获取路径内置函数
- [expand](./note/expand.md):获取路径内置函数

所以这里是加载`./core/vimrc`配置文件

- `<sfile>`:表示当前载入的配置文件
- `expand('<sfile>')`:扩展当前配置文件
- `:h`:当前文件的路径，去掉文件名部分，相对路径
- `fnamemodify(expand('<sfile>'),':h')`返回当前配置文件的相对路径

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






# 二、主题学习

这一章来总结整个架构每个知识点的学习。

## 1、插件配置

这里插件使用的是`dein`插件管理器来管理插件插件。

这里分析是在`thinkvim`中是如何使用`dein`的

- 初始脚本`init.vim`加载`core/vimrc`
- `core/vimrc`调用`etc#init()`
- `etc#init()`调用`etc#providers#dein#_init()`来完成`dein`脚本的初始化

重点来看下这个文件`autoload/etc/providers/dein.vim`

```css
function! etc#providers#dein#_init(config_paths) abort
	let l:cache_path = $DATA_PATH . '/dein'

	if has('vim_starting')
		" Use dein as a plugin manager
		let g:dein#auto_recache = 1
		let g:dein#install_max_processes = 16
		let g:dein#install_progress_type = 'echo'
		let g:dein#enable_notification = 1
		let g:dein#install_log_filename = $DATA_PATH . '/dein.log'

		" Add dein to vim's runtimepath
		if &runtimepath !~# '/dein.vim'
			let s:dein_dir = l:cache_path . '/repos/github.com/Shougo/dein.vim'
			" Clone dein if first-time setup
			if ! isdirectory(s:dein_dir)
				execute '!git clone https://github.com/Shougo/dein.vim' s:dein_dir
				if v:shell_error
					call s:error('dein installation has failed! is git installed?')
					finish
				endif
			endif

			execute 'set runtimepath+='.substitute(
				\ fnamemodify(s:dein_dir, ':p') , '/$', '', '')
		endif
	endif

	" Initialize dein.vim (package manager)
	if dein#load_state(l:cache_path)
		let l:rc = etc#_parse_config_files(a:config_paths)
		if empty(l:rc)
			call etc#util#error('Empty plugin list')
			return
		endif
		" Start propagating file paths and plugin presets
		call dein#begin(l:cache_path, extend([expand('<sfile>')], a:config_paths))
		for plugin in l:rc
			call dein#add(plugin['repo'], extend(plugin, {}, 'keep'))
		endfor

		" Add any local ./dev plugins
		if isdirectory(g:etc#vim_path.'/dev')
			call dein#local(g:etc#vim_path.'/dev', {'frozen': 1, 'merged': 0})
		endif
		call dein#end()
		call dein#save_state()

		" Update or install plugins if a change detected
		if dein#check_install()
			if ! has('nvim')
				set nomore
			endif
			call dein#install()
		endif
	endif

	filetype plugin indent on

	" Trigger source events, only when vim is starting
	if has('vim_starting')
	    syntax enable
        else
		call dein#call_hook('source')
		call dein#call_hook('post_source')
	endif
endfunction
```

`nvim`启动时将开始判断路径

```css
let s:dein_dir = l:cache_path . '/repos/github.com/Shougo/dein.vim'
```

判断这个路径是否存在

在我的电脑上是：`/home/pix/.cache/vim/dein/repos/github.com/Shougo/dein.vim`

就是`dein`的路径

接下来：

```css
if ! isdirectory(s:dein_dir)
	execute '!git clone https://github.com/Shougo/dein.vim' s:dein_dir
	if v:shell_error
		call s:error('dein installation has failed! is git installed?')
		finish
	endif
endif
```

如果路径不存在`clone dein仓库代码`到`s:dein_dir`目录

将`dein`安装目录加到`runtimepath`

```css
execute 'set runtimepath+='.substitute(
				\ fnamemodify(s:dein_dir, ':p') , '/$', '', '')
```

- 然后加载插件配置文件

- 执行dein加载插件流程

- 安装插件

- 其他操作：

  ```css
  " Trigger source events, only when vim is starting
  	if has('vim_starting')
  	    syntax enable
          else
  		call dein#call_hook('source')
  		call dein#call_hook('post_source')
  	endif
  ```

  参考：使用`YAML`管理插件：http://genkisugimoto.com/blog/manage-vim-plugins-via-yaml/ 

- 插件的配置文件：`core/dein/plugins.yaml`

- 插件的快捷键配置文件：`layers/+thinkvim/config.vim`

# 三、插件学习

插件的大概加载流程和配置文件目录已经知道了，接下来学习配置每个插件和学习每个插件的作用。

## 1、插件`denite.nvim`

`github:https://github.com/Shougo/denite.nvim `

看了下，这个插件是扩展用的，看作者的配置是用来处理`GoLang`的，暂时略过，先看别的

参考：<https://www.dazhuanlan.com/2019/10/17/5da842e59dc57/> 

## 2、插件fzf.vim

插件的github:https://github.com/junegunn/fzf.vim

首先是`core/dein/plugins.yaml`中的配置

```yaml
- repo: junegunn/fzf
  build: './install --all'
  merged: 0

- repo: junegunn/fzf.vim
  depends: fzf
  on_cmd: [Colors,Rg,Buffers]
  on_func: Fzf_dev
  hook_source: source  $VIM_PATH/layers/+completion/fzf/config.vim
```

这里hook_source:`hook_source: source  $VIM_PATH/layers/+completion/fzf/config.vim`

```css
"autocmd! FileType fzf
"autocmd  FileType fzf set laststatus=0 noshowmode noruler
  "\| autocmd BufLeave <buffer> set laststatus=0 showmode ruler
  "
let g:fzf_action = {
  \ 'ctrl-t': 'tab split',
  \ 'ctrl-x': 'split',
  \ 'ctrl-v': 'vsplit' }

" Customize fzf colors to match your color scheme
let g:fzf_colors =
\ { 'fg':      ['fg', 'Normal'],
  \ 'bg':      ['bg', '#5f5f87'],
  \ 'hl':      ['fg', 'Comment'],
  \ 'fg+':     ['fg', 'CursorLine', 'CursorColumn', 'Normal'],
  \ 'bg+':     ['bg', 'CursorLine', 'CursorColumn'],
  \ 'hl+':     ['fg', 'Statement'],
  \ 'info':    ['fg', 'PreProc'],
  \ 'border':  ['fg', 'Ignore'],
  \ 'prompt':  ['fg', 'Conditional'],
  \ 'pointer': ['fg', 'Exception'],
  \ 'marker':  ['fg', 'Keyword'],
  \ 'spinner': ['fg', 'Label'],
  \ 'header':  ['fg', 'Comment'] }

let g:fzf_commits_log_options = '--graph --color=always
  \ --format="%C(yellow)%h%C(red)%d%C(reset)
  \ - %C(bold green)(%ar)%C(reset) %s %C(blue)<%an>%C(reset)"'

"let $FZF_DEFAULT_COMMAND = 'ag --hidden -l -g ""'
" ripgrep
if executable('rg')
  let $FZF_DEFAULT_COMMAND = 'rg --files --hidden --follow --glob "!.git/*"'
  set grepprg=rg\ --vimgrep
  command! -bang -nargs=* Find call fzf#vim#grep('rg --column --line-number --no-heading --fixed-strings --ignore-case --hidden --follow --glob "!.git/*" --color "always" '.shellescape(<q-args>).'| tr -d "\017"', 1, <bang>0)
endif

let $FZF_DEFAULT_OPTS='--layout=reverse'
let g:fzf_layout = { 'window': 'call FloatingFZF()' }

function! FloatingFZF()
  let buf = nvim_create_buf(v:false, v:true)
  call setbufvar(buf, 'number', 'no')

  let height = float2nr(&lines/2)
  let width = float2nr(&columns - (&columns * 2 / 10))
  "let width = &columns
  let row = float2nr(&lines / 3)
  let col = float2nr((&columns - width) / 3)

  let opts = {
        \ 'relative': 'editor',
        \ 'row': row,
        \ 'col': col,
        \ 'width': width,
        \ 'height':height,
        \ }
  let win =  nvim_open_win(buf, v:true, opts)
  call setwinvar(win, '&number', 0)
  call setwinvar(win, '&relativenumber', 0)
endfunction

" Files + devicons
function! Fzf_dev()

let l:fzf_files_options = ' -m --bind ctrl-d:preview-page-down,ctrl-u:preview-page-up --preview "bat --color always --style numbers {2..}"'

  function! s:files()
    let l:files = split(system($FZF_DEFAULT_COMMAND), '\n')
    return s:prepend_icon(l:files)
  endfunction

  function! s:prepend_icon(candidates)
    let result = []
    for candidate in a:candidates
      let filename = fnamemodify(candidate, ':p:t')
      let icon = WebDevIconsGetFileTypeSymbol(filename, isdirectory(filename))
      call add(result, printf("%s %s", icon, candidate))
    endfor

    return result
  endfunction

  function! s:edit_file(items)
    let items = a:items
    let i = 1
    let ln = len(items)
    while i < ln
      let item = items[i]
      let parts = split(item, ' ')
      let file_path = get(parts, 1, '')
      let items[i] = file_path
      let i += 1
    endwhile
    call s:Sink(items)
  endfunction

  let opts = fzf#wrap({})
  let opts.source = <sid>files()
  let s:Sink = opts['sink*']
  let opts['sink*'] = function('s:edit_file')
  let opts.options .= l:fzf_files_options
  call fzf#run(opts)
endfunction

```

## 3、插件which-key

Github:https://github.com/liuchengxu/vim-which-key
原作者博客：https://www.jianshu.com/p/e47f7ec27cea

首先是`core/dein/plugins.yaml`中的配置

```yaml
- repo: liuchengxu/vim-which-key
  on_cmd: [Whichkey, Whichkey!]
  hook_source: source  $VIM_PATH/layers/+tools/whichkey/config.vim
  hook_post_source: |
        call which_key#register('<Space>', 'g:which_key_map')
        call which_key#register(';', 'g:which_key_localmap')
        call which_key#register(']', 'g:which_key_rsbgmap')
        call which_key#register('[', 'g:which_key_lsbgmap')
```

插件配置文件：` $VIM_PATH/layers/+tools/whichkey/config.vim`

```css
let g:which_key_map =  {}
let g:which_key_map = {
      \ 'name' : '+ThinkVim root ' ,
      \ '1' : 'select window-1'      ,
      \ '2' : 'select window-2'      ,
      \ '3' : 'select window-3'      ,
      \ '4' : 'select window-4'      ,
      \ '5' : 'select window-5'      ,
      \ '6' : 'select window-6'      ,
      \ '7' : 'select window-7'      ,
      \ '8' : 'select window-8'      ,
      \ '9' : 'select window-9'      ,
      \ '0' : 'select window-10'      ,
      \ 'a' : {
            \ 'name' : '+coc-code-action',
            \ 'c' : 'code action',
            \ },
      \ 'b' : {
            \ 'name' : '+buffer',
            \ 'b' : 'buffer list',
            \ 'c' : 'keep current buffer',
            \ 'o' : 'keep input buffer',
            \ },
      \ 'e' : 'open file explorer' ,
      \ '-' : 'choose window by {prompt char}' ,
      \ 'd' : 'search cursor word on Dash.app' ,
      \ 'G' : 'distraction free writing' ,
      \ 'F' : 'find current file' ,
      \ 'f' : {
            \ 'name' : '+search {files cursorword word outline}',
            \ 'f' : 'find file',
            \ 'r' : 'search {word}',
            \ 'c' : 'change colorscheme',
            \ 'w' : 'search cursorword',
            \ 'v' : 'search outline',
            \ },
      \ 'm' : 'open mundotree' ,
      \ 'w' : 'save file',
      \ 'j' : 'open coc-explorer',
      \ 's' : 'open startify screen',
      \ 'p' : 'edit pluginsconfig {filename}',
      \ 'x' : 'coc cursors operate',
      \ 'g'  :{
                \'name':'+git-operate',
                \ 'd'    : 'Gdiff',
                \ 'c'    : 'Gcommit',
                \ 'b'    : 'Gblame',
                \ 'B'    : 'Gbrowse',
                \ 'S'    : 'Gstatus',
                \ 'p'    : 'git push',
                \ 'l'    : 'GitLogAll',
                \ 'h'    : 'GitBranch',
                \},
      \ 'c'    : {
              \ 'name' : '+coc list' ,
              \ 'a'    : 'coc CodeActionSelected',
              \ 'd'    : 'coc Diagnostics',
              \ 'c'    : 'coc Commands',
              \ 'e'    : 'coc Extensions',
              \ 'j'    : 'coc Next',
              \ 'k'    : 'coc Prev',
              \ 'o'    : 'coc OutLine',
              \ 'r'    : 'coc Resume',
              \ 'n'    : 'coc Rename',
              \ 's'    : 'coc Isymbols',
              \ 'g'    : 'coc Gitstatus',
              \ 'f'    : 'coc Format',
              \ 'm'    : 'coc search word to multiple cursors',
              \ },
      \ 'q' : {
            \ 'name' : '+coc-quickfix',
            \ 'f' : 'coc fixcurrent',
            \ },
      \ 't' : {
            \ 'name' : '+tab-operate',
            \ 'n' : 'new tab',
            \ 'e' : 'edit tab',
            \ 'm' : 'move tab',
            \ },
      \ }
let g:which_key_map[' '] = {
      \ 'name' : '+easymotion-jumpto-word ' ,
      \ 'b' : ['<plug>(easymotion-b)' , 'beginning of word backward'],
      \ 'f' : ['<plug>(easymotion-f)' , 'find {char} to the left'],
      \ 'w' : ['<plug>(easymotion-w)' , 'beginning of word forward'],
      \ }

let g:which_key_localmap ={
      \ 'name' : '+LocalLeaderKey'  ,
      \ 'v'    : 'open vista show outline',
      \ 'r'    : 'quick run',
      \ 'm'    : 'toolkit Menu',
      \ 'g' : {
            \ 'name' : '+golang-toolkit',
            \ 'i'    : 'go impl',
            \ 'd'    : 'go describe',
            \ 'c'    : 'go callees',
            \ 'C'    : 'go callers',
            \ 's'    : 'go callstack',
            \ },
      \ }

let g:which_key_rsbgmap = {
      \ 'name' : '+RightSquarebrackets',
      \ 'a'    : 'ale nextwarp',
      \ 'c'    : 'coc nextdiagnostics',
      \ 'b'    : 'next buffer',
      \ 'g'    : 'coc gitnextchunk',
      \ ']'    : 'jump prefunction-golang',
      \ }


let g:which_key_lsbgmap = {
      \ 'name' : '+LeftSquarebrackets',
      \ 'a'    : 'ale prewarp',
      \ 'c'    : 'coc prediagnostics',
      \ 'b'    : 'pre buffer',
      \ 'g'    : 'coc gitprevchunk',
      \ '['    : 'jump nextfunction-golang',
      \ }

let s:current_colorscheme = get(g:,"colors_name","")
if  s:current_colorscheme == "base16-default-dark"
    highlight WhichKeySeperator guibg=NONE ctermbg=NONE guifg=#a1b56c ctermfg=02
endif

```

## 4、vim-gutentags

这个是用来自动生成tags的插件

Github:https://github.com/ludovicchabant/vim-gutentags

参考阅读：https://www.cnblogs.com/pengdonglin137/articles/10202606.html

首先是core/dein/plugins.yaml中的配置

```yaml
- repo: ludovicchabant/vim-gutentags
  if: executable('ctags')
  on_path: .*
  hook_source: source  $VIM_PATH/layers/+tools/vim-gutentags/config.vim
```

配置文件` $VIM_PATH/layers/+tools/vim-gutentags/config.vim`

```css
let g:gutentags_cache_dir = $DATA_PATH . '/tags'
let g:gutentags_project_root = ['.root', '.git', '.svn', '.hg', '.project','go.mod','/usr/local']
let g:gutentags_generate_on_write = 1
let g:gutentags_generate_on_missing = 1
let g:gutentags_generate_on_new = 0
let g:gutentags_exclude_filetypes = [ 'defx', 'denite', 'vista', 'magit' ]
let g:gutentags_ctags_extra_args = ['--output-format=e-ctags']
let g:gutentags_ctags_exclude = ['*.json', '*.js', '*.ts', '*.jsx', '*.css', '*.less', '*.sass', '*.go', '*.dart', 'node_modules', 'dist', 'vendor']
```

## 5、defx.nvim

github:https://github.com/Shougo/defx.nvim

这个插件是用来浏览文件目录的，取代NERDTree

首先是core/dein/plugins.yaml中的配置

```yaml
- repo: Shougo/defx.nvim
  on_cmd: Defx
  hook_source: source  $VIM_PATH/layers/+ui/defx/config.vim
```

配置文件：`$VIM_PATH/layers/+ui/defx/config.vim`

```css
" Defx
call defx#custom#option('_', {
	\ 'columns': 'indent:git:icons:filename',
	\ 'winwidth': 30,
	\ 'split': 'vertical',
	\ 'direction': 'topleft',
	\ 'show_ignored_files': 0,
	\ })


" Events
" ---

augroup user_plugin_defx
	autocmd!

	" autocmd DirChanged * call s:defx_refresh_cwd(v:event)

	" Delete defx if it's the only buffer left in the window
	" autocmd WinEnter * if &filetype == 'defx' && winnr('$') == 1 | bd | endif

	" Move focus to the next window if current buffer is defx
	autocmd TabLeave * if &filetype == 'defx' | wincmd w | endif

	autocmd TabClosed * call s:defx_close_tab(expand('<afile>'))

	" Define defx window mappings
	autocmd FileType defx call s:defx_mappings()

augroup END

" Internal functions
" ---

function! s:defx_close_tab(tabnr)
	" When a tab is closed, find and delete any associated defx buffers
	for l:nr in range(1, bufnr('$'))
		let l:defx = getbufvar(l:nr, 'defx')
		if empty(l:defx)
			continue
		endif
		let l:context = get(l:defx, 'context', {})
		if get(l:context, 'buffer_name', '') ==# 'tab' . a:tabnr
			silent! execute 'bdelete '.l:nr
			break
		endif
	endfor
endfunction

function! s:defx_toggle_tree() abort
	" Open current file, or toggle directory expand/collapse
	if defx#is_directory()
		return defx#do_action('open_or_close_tree')
	endif
	return defx#do_action('multi', ['drop'])
endfunction

function! s:defx_refresh_cwd(event)
	" Automatically refresh opened Defx windows when changing working-directory
	let l:cwd = get(a:event, 'cwd', '')
	let l:scope = get(a:event, 'scope', '')   " global, tab, window
	let l:is_opened = bufwinnr('defx') > -1
	if ! l:is_opened || empty(l:cwd) || empty(l:scope)
		return
	endif

	" Abort if Defx is already on the cwd triggered (new files trigger this)
	let l:paths = get(getbufvar('defx', 'defx', {}), 'paths', [])
	if index(l:paths, l:cwd) >= 0
		return
	endif

	let l:tab = tabpagenr()
	call execute(printf('Defx -buffer-name=tab%s %s', l:tab, l:cwd))
	wincmd p
endfunction

function! s:jump_dirty(dir) abort
	" Jump to the next position with defx-git dirty symbols
	let l:icons = get(g:, 'defx_git_indicators', {})
	let l:icons_pattern = join(values(l:icons), '\|')

	if ! empty(l:icons_pattern)
		let l:direction = a:dir > 0 ? 'w' : 'bw'
		return search(printf('\(%s\)', l:icons_pattern), l:direction)
	endif
endfunction

function! s:defx_mappings() abort
	" Defx window keyboard mappings
	setlocal signcolumn=no

	nnoremap <silent><buffer><expr> <CR>  defx#do_action('drop')
	nnoremap <silent><buffer><expr> l     <SID>defx_toggle_tree()
	nnoremap <silent><buffer><expr> h     defx#async_action('cd', ['..'])
	nnoremap <silent><buffer><expr> st    defx#do_action('multi', [['drop', 'tabnew'], 'quit'])
	nnoremap <silent><buffer><expr> s     defx#do_action('open', 'botright vsplit')
	nnoremap <silent><buffer><expr> i     defx#do_action('open', 'botright split')
	nnoremap <silent><buffer><expr> P     defx#do_action('open', 'pedit')
	nnoremap <silent><buffer><expr> K     defx#do_action('new_directory')
	nnoremap <silent><buffer><expr> N     defx#do_action('new_multiple_files')
	nnoremap <silent><buffer><expr> dd    defx#do_action('remove_trash')
	nnoremap <silent><buffer><expr> r     defx#do_action('rename')
	nnoremap <silent><buffer><expr> x     defx#do_action('execute_system')
	nnoremap <silent><buffer><expr> .     defx#do_action('toggle_ignored_files')
	nnoremap <silent><buffer><expr> yy    defx#do_action('yank_path')
	nnoremap <silent><buffer><expr> ~     defx#async_action('cd')
	nnoremap <silent><buffer><expr> q     defx#do_action('quit')
	nnoremap <silent><buffer><expr> <Tab> winnr('$') != 1 ?
		\ ':<C-u>wincmd w<CR>' :
		\ ':<C-u>Defx -buffer-name=temp -split=vertical<CR>'

	nnoremap <silent><buffer>       [g :<C-u>call <SID>jump_dirty(-1)<CR>
	nnoremap <silent><buffer>       ]g :<C-u>call <SID>jump_dirty(1)<CR>

	nnoremap <silent><buffer><expr><nowait> \  defx#do_action('cd', getcwd())
	nnoremap <silent><buffer><expr><nowait> &  defx#do_action('cd', getcwd())
	nnoremap <silent><buffer><expr><nowait> c  defx#do_action('copy')
	nnoremap <silent><buffer><expr><nowait> m  defx#do_action('move')
	nnoremap <silent><buffer><expr><nowait> p  defx#do_action('paste')

	nnoremap <silent><buffer><expr><nowait> <Space>
		\ defx#do_action('toggle_select') . 'j'

	nnoremap <silent><buffer><expr> '      defx#do_action('toggle_select') . 'j'
	nnoremap <silent><buffer><expr> *      defx#do_action('toggle_select_all')
	nnoremap <silent><buffer><expr> <C-r>  defx#do_action('redraw')
	nnoremap <silent><buffer><expr> <C-g>  defx#do_action('print')

	nnoremap <silent><buffer><expr> S  defx#do_action('toggle_sort', 'Time')
	nnoremap <silent><buffer><expr> C
		\ defx#do_action('toggle_columns', 'indent:mark:filename:type:size:time')

	" Tools
	nnoremap <silent><buffer><expr> gx  defx#async_action('execute_system')
	nnoremap <silent><buffer><expr> gd  defx#async_action('multi', ['drop', ['call', '<SID>git_diff']])
	nnoremap <silent><buffer><expr> gl  defx#async_action('call', '<SID>explorer')
	nnoremap <silent><buffer><expr> gr  defx#do_action('call', '<SID>grep')
	nnoremap <silent><buffer><expr> gf  defx#do_action('call', '<SID>find_files')
	nnoremap <silent><buffer><expr> w   defx#async_action('call', '<SID>toggle_width')
endfunction

" TOOLS
" ---

function! s:git_diff(context) abort
	execute 'GdiffThis'
endfunction

function! s:find_files(context) abort
	" Find files in parent directory with Denite
	let l:target = a:context['targets'][0]
	let l:parent = fnamemodify(l:target, ':h')
	silent execute 'wincmd w'
	silent execute 'Denite file/rec:'.l:parent
endfunction

function! s:grep(context) abort
	" Grep in parent directory with Denite
	let l:target = a:context['targets'][0]
	let l:parent = fnamemodify(l:target, ':h')
	silent execute 'wincmd w'
	silent execute 'Denite grep:'.l:parent
endfunction

function! s:toggle_width(context) abort
	" Toggle between defx window width and longest line
	let l:max = 0
	let l:original = a:context['winwidth']
	for l:line in range(1, line('$'))
		let l:len = len(getline(l:line))
		if l:len > l:max
			let l:max = l:len
		endif
	endfor
	execute 'vertical resize ' . (l:max == winwidth('.') ? l:original : l:max)
endfunction

function! s:explorer(context) abort
	" Open file-explorer split with tmux
	let l:explorer = s:find_file_explorer()
	if empty('$TMUX') || empty(l:explorer)
		return
	endif
	let l:target = a:context['targets'][0]
	let l:parent = fnamemodify(l:target, ':h')
	let l:cmd = 'split-window -p 30 -c ' . l:parent . ' ' . l:explorer
	silent execute '!tmux ' . l:cmd
endfunction

function! s:find_file_explorer() abort
	" Detect terminal file-explorer
	let s:file_explorer = get(g:, 'terminal_file_explorer', '')
	if empty(s:file_explorer)
		for l:explorer in ['lf', 'hunter', 'ranger', 'vifm']
			if executable(l:explorer)
				let s:file_explorer = l:explorer
				break
			endif
		endfor
	endif
	return s:file_explorer
endfunction
```

## 6、vim-easygit

Github:https://github.com/neoclide/vim-easygit

用来vim支持git的插件。

首先是core/dein/plugins.yaml中的配置

```yaml
- repo: chemzqm/vim-easygit
  on_cmd: [Gcd, Glcd, Gcommit, Gblame, Gstatus, Gdiff, Gbrowse, Gstatus, Gpush]
  hook_source: let g:easygit_enable_command = 1
```

## 7、defx-git

Github:https://github.com/kristijanhusak/defx-git

这个插件是用来给`defx`来提供git状态支持的

首先是core/dein/plugins.yaml中的配置

```yaml
- repo: kristijanhusak/defx-git
  on_source: defx.nvim
  hook_source: source  $VIM_PATH/layers/+ui/defx/+defx-git.vim
```

配置文件：`$VIM_PATH/layers/+ui/defx/+defx-git.vim`

```yaml

let g:defx_git#indicators = {
	\ 'Modified'  : '•',
	\ 'Staged'    : '✚',
	\ 'Untracked' : 'ᵁ',
	\ 'Renamed'   : '≫',
	\ 'Unmerged'  : '≠',
	\ 'Ignored'   : 'ⁱ',
	\ 'Deleted'   : '✖',
	\ 'Unknown'   : '⁇'
	\ }

```











# 四、快捷键

## 1、fzf

| 按键                                         | 映射                                               | 功能             |
| -------------------------------------------- | -------------------------------------------------- | ---------------- |
| <kbd>LEADER</kbd> + <kbd>f</kbd><kbd>c</kbd> | `nnoremap <silent> <leader>fc :Colors<CR>`         | 选择颜色主题     |
| <kbd>LEADER</kbd> + <kbd>b</kbd><kbd>b</kbd> | `nnoremap <silent> <leader>bb :Buffers<CR>`        | 选择buffer       |
| <kbd>LEADER</kbd> + <kbd>f</kbd><kbd>f</kbd> | `nnoremap <silent> <leader>ff :call Fzf_dev()<CR>` | 搜索项目中的文件 |
| <kbd>LEADER</kbd> + <kbd>f</kbd><kbd>r</kbd> | `nnoremap <silent> <leader>fr :Rg<CR>`             | 执行Rg搜索       |
| <kbd>LEADER</kbd> + <kbd>f</kbd><kbd>w</kbd> | `nnoremap <silent> <leader>fw :Rg <C-R><C-W><CR>`  | 搜索光标所在词   |

## 2、which-key

这个插件定义了很多快捷键，有时间逐个试下，总结一下，这个插件本身的作用就是引导用户来使用快捷键。

四个引导键：

- `<LEADER>`
- `<LOCALLEADER>`
- `[`
- `]`

## 3、defx

文件浏览器defx的快捷键

| 按键                           | 映射                                                         | 功能                       |
| ------------------------------ | ------------------------------------------------------------ | -------------------------- |
| <kbd>Enter</kbd>               | `nnoremap <silent><buffer><expr> <CR>  defx#do_action('drop')` | 进入目录/打开文件          |
| <kbd>l</kbd>                   | `nnoremap <silent><buffer><expr> l     <SID>defx_toggle_tree()` | 展开目录                   |
| <kbd>h</kbd>                   | `nnoremap <silent><buffer><expr> h     defx#async_action('cd', ['..'])` | 回到上级目录               |
| <kbd>s</kbd><kbd>t</kbd>       | `nnoremap <silent><buffer><expr> st    defx#do_action('multi', [['drop', 'tabnew'], 'quit'])` | 新标签打开文件，关闭defx   |
| <kbd>s</kbd>                   | `nnoremap <silent><buffer><expr> s     defx#do_action('open', 'botright vsplit')` | 垂直分屏打开文件，关闭defx |
| <kbd>i</kbd>                   | `nnoremap <silent><buffer><expr> i     defx#do_action('open', 'botright split')` | 纵向分屏打开文件，关闭defx |
| <kbd>P</kbd>                   | `nnoremap <silent><buffer><expr> P     defx#do_action('open', 'pedit')` | 在左上角打开，不知道有啥用 |
| <kbd>K</kbd>                   | `nnoremap <silent><buffer><expr> K     defx#do_action('new_directory')` | 创建一个目录               |
| <kbd>N</kbd>                   | `nnoremap <silent><buffer><expr> N     defx#do_action('new_multiple_files')` | 复制黏贴文件               |
| <kbd>d</kbd><kbd>d</kbd>       | `nnoremap <silent><buffer><expr> dd    defx#do_action('remove_trash')` | 删除文件                   |
| <kbd>r</kbd>                   | `nnoremap <silent><buffer><expr> r     defx#do_action('rename')` | 重命名                     |
| <kbd>x</kbd>                   | `nnoremap <silent><buffer><expr> x     defx#do_action('execute_system')` | 执行文件                   |
| <kbd>.</kbd>                   | `nnoremap <silent><buffer><expr> .     defx#do_action('toggle_ignored_files')` | 显示隐藏文件               |
| <kbd>y</kbd><kbd>y</kbd>       | `nnoremap <silent><buffer><expr> yy    defx#do_action('yank_path')` | 复制                       |
| <kbd>～</kbd>                  | `nnoremap <silent><buffer><expr> ~     defx#async_action('cd')` | 回到宿主目录               |
| <kbd>q</kbd>                   | `nnoremap <silent><buffer><expr> q     defx#do_action('quit')` | 退出defx                   |
| <kbd>TAB</kbd>                 | `nnoremap <silent><buffer><expr> <Tab> winnr('$') != 1 ?<br/>		\ ':<C-u>wincmd w<CR>' :<br/>		\ ':<C-u>Defx -buffer-name=temp -split=vertical<CR>'` | 切换窗口                   |
| <kbd>[</kbd><kbd>g</kbd>       | `nnoremap <silent><buffer>       [g :<C-u>call <SID>jump_dirty(-1)<CR>` | 未知                       |
| <kbd>]</kbd><kbd>g</kbd>       | `nnoremap <silent><buffer>       ]g :<C-u>call <SID>jump_dirty(1)<CR>` | 未知                       |
| <kbd>\\</kbd>                  | `nnoremap <silent><buffer><expr><nowait> \  defx#do_action('cd', getcwd())` | 切换到工作目录             |
| <kbd>&</kbd>                   | `nnoremap <silent><buffer><expr><nowait> &  defx#do_action('cd', getcwd())` | 同<kbd>\\</kbd>            |
| <kbd>c</kbd>                   | `nnoremap <silent><buffer><expr><nowait> c  defx#do_action('copy')` | 拷贝                       |
| <kbd>m</kbd>                   | `nnoremap <silent><buffer><expr><nowait> m  defx#do_action('move')` | 剪切\移动                  |
| <kbd>p</kbd>                   | `nnoremap <silent><buffer><expr><nowait> p  defx#do_action('paste')` | 粘贴                       |
| <kbd>SPACE</kbd>               | `nnoremap <silent><buffer><expr><nowait> <Space><br/>		\ defx#do_action('toggle_select') . 'j'` | 选择                       |
| <kbd>'</kbd>                   | `nnoremap <silent><buffer><expr> '      defx#do_action('toggle_select') . 'j'` | 选择                       |
| <kbd>*</kbd>                   | `nnoremap <silent><buffer><expr> *      defx#do_action('toggle_select_all')` | 全选                       |
| <kbd>Ctrl</kbd> + <kbd>r</kbd> | `nnoremap <silent><buffer><expr> <C-r>  defx#do_action('redraw')` | 重绘                       |
| <kbd>Ctrl</kbd> + <kbd>g</kbd> | `nnoremap <silent><buffer><expr> <C-g>  defx#do_action('print')` | 打印当前文件全路径         |
| <kbd>S</kbd>                   | `nnoremap <silent><buffer><expr> S  defx#do_action('toggle_sort', 'Time')` | 按时间排序                 |
| <kbd>C</kbd>                   | `nnoremap <silent><buffer><expr> C<br/>		\ defx#do_action('toggle_columns', 'indent:mark:filename:type:size:time')` | 显示样式                   |
| <kbd>g</kbd><kbd>x</kbd>       | `nnoremap <silent><buffer><expr> gx  defx#async_action('execute_system')` | 异步执行                   |
| <kbd>g</kbd><kbd>d</kbd>       | `nnoremap <silent><buffer><expr> gd  defx#async_action('multi', ['drop', ['call', '<SID>git_diff']])` | git diff                   |
| <kbd>g</kbd><kbd>l</kbd>       | `nnoremap <silent><buffer><expr> gl  defx#async_action('call', '<SID>explorer')` | 类似于ranger的方式浏览     |
| <kbd>g</kbd><kbd>r</kbd>       | `nnoremap <silent><buffer><expr> gr  defx#do_action('call', '<SID>grep')` | 过滤搜索                   |
| <kbd>g</kbd><kbd>f</kbd>       | `nnoremap <silent><buffer><expr> gf  defx#do_action('call', '<SID>find_files')` | 列出所有文件               |
| <kbd>w</kbd>                   | `nnoremap <silent><buffer><expr> w   defx#async_action('call', '<SID>toggle_width')` | 文件浏览器defx变宽显示     |





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