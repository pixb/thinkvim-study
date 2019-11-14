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

plugins.yaml

```yaml
- repo: Shougo/denite.nvim
  on_cmd: Denite
  depends: vim-devicons
  hook_source: |
        source  $VIM_PATH/layers/+completion/denite/config.vim
        source  $VIM_PATH/layers/+completion/denite/+denite_menu.vim
```

- `$VIM_PATH/layers/+completion/denite/config.vim`

  ```css
  call denite#custom#option('_', {
  		\ 'cached_filter': v:true,
  		\ 'cursor_shape': v:true,
  		\ 'cursor_wrap': v:true,
  		\ 'highlight_filter_background': 'DeniteFilter',
  		\ 'highlight_matched_char': 'Underlined',
  		\ 'matchers': 'matcher/fuzzy',
  		\ 'prompt': 'λ ',
  		\ 'split': 'floating',
  		\ 'start_filter': v:false,
  		\ 'statusline': v:false,
  		\ })
  function! s:denite_detect_size() abort
      let s:denite_winheight = 20
      let s:denite_winrow = &lines > s:denite_winheight ? (&lines - s:denite_winheight) / 2 : 0
      let s:denite_winwidth = &columns > 240 ? &columns / 2 : 120
      let s:denite_wincol = &columns > s:denite_winwidth ? (&columns - s:denite_winwidth) / 2 : 0
      call denite#custom#option('_', {
           \ 'wincol': s:denite_wincol,
           \ 'winheight': s:denite_winheight,
           \ 'winrow': s:denite_winrow,
           \ 'winwidth': s:denite_winwidth,
           \ })
    endfunction
     augroup denite-detect-size
      autocmd!
      autocmd VimResized * call <SID>denite_detect_size()
    augroup END
    call s:denite_detect_size()
  
  
  call denite#custom#option('search', { 'start_filter': 0, 'no_empty': 1 })
  call denite#custom#option('list', { 'start_filter': 0 })
  call denite#custom#option('jump', { 'start_filter': 0 })
  call denite#custom#option('git', { 'start_filter': 0 })
  call denite#custom#option('mpc', { 'winheight': 20 })
  
  
  " MATCHERS
  " Default is 'matcher/fuzzy'
  call denite#custom#source('tag', 'matchers', ['matcher/substring'])
  call denite#custom#source('file/rec', 'matchers', ['matcher/fuzzy'])
  
  if has('nvim') && &runtimepath =~# '\/cpsm'
  	call denite#custom#source(
  		\ 'buffer,file_mru,file/old,file/rec,grep,mpc,line,neoyank',
  		\ 'matchers', ['matcher/cpsm', 'matcher/fuzzy'])
  endif
  
  
  " CONVERTERS
  " Default is none
  call denite#custom#source(
  	\ 'buffer,file_mru,file/old,file/rec,directory/rec,directory_mru',
  	\ 'converters', ['devicons_denite_converter','converter_relative_word'])
  
  " FIND and GREP COMMANDS
  if executable('ag')
  	" The Silver Searcher
  	call denite#custom#var('file/rec', 'command',
  		\ ['ag', '-U', '--hidden', '--follow', '--nocolor', '--nogroup', '-g', ''])
  
  	" Setup ignore patterns in your .agignore file!
  	" https://github.com/ggreer/the_silver_searcher/wiki/Advanced-Usage
  
  	call denite#custom#var('grep', 'command', ['ag'])
  	call denite#custom#var('grep', 'recursive_opts', [])
  	call denite#custom#var('grep', 'pattern_opt', [])
  	call denite#custom#var('grep', 'separator', ['--'])
  	call denite#custom#var('grep', 'final_opts', [])
  	call denite#custom#var('grep', 'default_opts',
  		\ [ '--skip-vcs-ignores', '--vimgrep', '--smart-case', '--hidden' ])
  
  elseif executable('ack')
  	" Ack command
  	call denite#custom#var('grep', 'command', ['ack'])
  	call denite#custom#var('grep', 'recursive_opts', [])
  	call denite#custom#var('grep', 'pattern_opt', ['--match'])
  	call denite#custom#var('grep', 'separator', ['--'])
  	call denite#custom#var('grep', 'final_opts', [])
  	call denite#custom#var('grep', 'default_opts',
  			\ ['--ackrc', $HOME.'/.config/ackrc', '-H',
  			\ '--nopager', '--nocolor', '--nogroup', '--column'])
  
  elseif executable('rg')
  	" Ripgrep
    call denite#custom#var('file/rec', 'command',
          \ ['rg', '--files', '--glob', '!.git'])
    call denite#custom#var('grep', 'command', ['rg', '--threads', '1'])
    call denite#custom#var('grep', 'recursive_opts', [])
    call denite#custom#var('grep', 'final_opts', [])
    call denite#custom#var('grep', 'separator', ['--'])
    call denite#custom#var('grep', 'default_opts',
          \ ['-i', '--vimgrep', '--no-heading'])
  endif
  
  
  " KEY MAPPINGS
  autocmd FileType denite call s:denite_settings()
  function! s:denite_settings() abort
  	highlight! link CursorLine Visual
  	nnoremap <silent><buffer><expr> <CR> denite#do_map('do_action')
  	nnoremap <silent><buffer><expr> i    denite#do_map('open_filter_buffer')
  	nnoremap <silent><buffer><expr> d    denite#do_map('do_action', 'delete')
  	nnoremap <silent><buffer><expr> p    denite#do_map('do_action', 'preview')
  	nnoremap <silent><buffer><expr> st   denite#do_map('do_action', 'tabopen')
  	nnoremap <silent><buffer><expr> sv   denite#do_map('do_action', 'vsplit')
  	nnoremap <silent><buffer><expr> si   denite#do_map('do_action', 'split')
  	nnoremap <silent><buffer><expr> '    denite#do_map('quick_move')
  	nnoremap <silent><buffer><expr> q    denite#do_map('quit')
  	nnoremap <silent><buffer><expr> r    denite#do_map('redraw')
  	nnoremap <silent><buffer><expr> yy   denite#do_map('do_action', 'yank')
  	nnoremap <silent><buffer><expr> <Esc>   denite#do_map('quit')
  	nnoremap <silent><buffer><expr> <C-u>   denite#do_map('restore_sources')
  	nnoremap <silent><buffer><expr> <C-f>   denite#do_map('do_action', 'defx')
  	nnoremap <silent><buffer><expr> <C-x>   denite#do_map('choose_action')
  	nnoremap <silent><buffer><expr><nowait> <Space> denite#do_map('toggle_select').'j'
  endfunction
  
  autocmd FileType denite-filter call s:denite_filter_settings()
  function! s:denite_filter_settings() abort
  	nnoremap <silent><buffer><expr> <Esc>  denite#do_map('quit')
  	" inoremap <silent><buffer><expr> <Esc>  denite#do_map('quit')
  	nnoremap <silent><buffer><expr> q      denite#do_map('quit')
  	imap <silent><buffer> <C-c> <Plug>(denite_filter_quit)
  	"inoremap <silent><buffer><expr> <C-c>  denite#do_map('quit')
  	nnoremap <silent><buffer><expr> <C-c>  denite#do_map('quit')
  	inoremap <silent><buffer>       kk     <Esc><C-w>p
  	nnoremap <silent><buffer>       kk     <C-w>p
  	inoremap <silent><buffer>       jj     <Esc><C-w>p
  	nnoremap <silent><buffer>       jj     <C-w>p
  endfunction
  
  " vim: set ts=3 sw=2 tw=80 noet :
  ```

- `$VIM_PATH/layers/+completion/denite/+denite_menu.vim`

  ```yaml
  let s:menus = {}
  
  let s:menus.dein = { 'description': '⚔️  Plugin management' }
  let s:menus.dein.command_candidates = [
    \   ['🐬 Dein: Plugins update       🔸', 'call dein#update()'],
    \   ['🐬 Dein: Plugins List         🔸', 'Denite dein'],
    \   ['🐬 Dein: RecacheRuntimePath   🔸', 'call dein#recache_runtimepath()'],
    \   ['🐬 Dein: Update log           🔸', 'echo dein#get_updates_log()'],
    \   ['🐬 Dein: Log                  🔸', 'echo dein#get_log()'],
    \ ]
  
  let s:menus.project = { 'description': '🛠  Project & Structure' }
  let s:menus.project.command_candidates = [
    \   ['🐳 File Explorer        🔸<Leader>e',        'Defx -resume -toggle -buffer-name=tab`tabpagenr()`<CR>'],
    \   ['🐳 Outline              🔸<LocalLeader>t',   'TagbarToggle'],
    \   ['🐳 Git Status           🔸<LocalLeader>gs',  'Denite gitstatus'],
    \   ['🐳 Mundo Tree           🔸<Leader>m',  'MundoToggle'],
    \ ]
  
  let s:menus.files = { 'description': '📁 File tools' }
  let s:menus.files.command_candidates = [
    \   ['📂 Denite: Find in files…    🔹 ',  'Denite grep:.'],
    \   ['📂 Denite: Find files        🔹 ',  'Denite file/rec'],
    \   ['📂 Denite: Buffers           🔹 ',  'Denite buffer'],
    \   ['📂 Denite: MRU               🔹 ',  'Denite file/old'],
    \   ['📂 Denite: Line              🔹 ',  'Denite line'],
    \ ]
  
  let s:menus.tools = { 'description': '⚙️  Dev Tools' }
  let s:menus.tools.command_candidates = [
    \   ['🐠 Git commands       🔹', 'Git'],
    \   ['🐠 Git log            🔹', 'Denite gitlog:all'],
    \   ['🐠 Goyo               🔹', 'Goyo'],
    \   ['🐠 Tagbar             🔹', 'TagbarToggle'],
    \   ['🐠 File explorer      🔹', 'Defx -resume -toggle -buffer-name=tab`tabpagenr()`<CR>'],
    \ ]
  
  let s:menus.config = { 'description': '🔧 Zsh Tmux Configuration' }
  let s:menus.config.file_candidates = [
    \   ['🐠 Zsh Configurationfile            🔸', '~/.zshrc'],
    \   ['🐠 Tmux Configurationfile           🔸', '~/.tmux.conf'],
    \ ]
  
  let s:menus.thinkvim = {'description': '💎 ThinkVim Configuration files'}
  let s:menus.thinkvim.file_candidates = [
    \   ['🐠 MainVimrc          settings: vimrc               🔹', $VIMPATH.'/core/vimrc'],
    \   ['🐠 Initial            settings: init.vim            🔹', $VIMPATH.'/core/init.vim'],
    \   ['🐠 General            settings: general.vim         🔹', $VIMPATH.'/core/general.vim'],
    \   ['🐠 DeinConfig         settings: deinrc.vim          🔹', $VIMPATH.'/core/deinrc.vim'],
    \   ['🐠 FileTypes          settings: filetype.vim        🔹', $VIMPATH.'/core/filetype.vim'],
    \   ['🐠 Installed       LoadPlugins: plugins.yaml        🔹', $VIMPATH.'/core/dein/plugins.yaml'],
    \   ['🐠 Installed      LocalPlugins: local_plugins.yaml  🔹', $VIMPATH.'/core/dein/local_plugins.yaml'],
    \   ['🐠 Global   Key    Vimmappings: mappings.vim        🔹', $VIMPATH.'/core/mappings.vim'],
    \   ['🐠 Global   Key Pluginmappings: Pluginmappings      🔹', $VIMPATH.'/core/plugins/allkey.vim'],
    \ ]
  
  call denite#custom#var('menu', 'menus', s:menus)
  
  "let s:menus.sessions = { 'description': 'Sessions' }
  "let s:menus.sessions.command_candidates = [
    "\   ['▶ Restore session │ ;s', 'Denite session'],
    "\   ['▶ Save session…   │', 'Denite session/new'],
    "\ ]
  ```

  

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

## 8、defx-icons

Github:https://github.com/kristijanhusak/defx-icons

plugins.yaml

```yaml
- repo: kristijanhusak/defx-icons
  on_source: defx.nvim
```

## 9、magit.vim

更好的查看提交记录

github:https://github.com/taigacute/magit.vim

plugins.yaml

```yaml
- { repo: taigacute/magit.vim, on_cmd: Magit }
```

## 10、gina.vim

另一个git工具

github:https://github.com/lambdalisue/gina.vim

plugins.yaml

```yaml
- { repo: lambdalisue/gina.vim, on_cmd: Gina }
```

## 11、vim-easymotion

快速移动工具

github:https://github.com/easymotion/vim-easymotion

plugins.yaml

```yaml
- repo: easymotion/vim-easymotion
  on_map: { n: <Plug> }
  hook_source: |
        let g:EasyMotion_do_mapping = 0
        let g:EasyMotion_prompt = 'Jump to → '
        let g:EasyMotion_keys = 'fjdkswbeoavn'
        let g:EasyMotion_smartcase = 1
        let g:EasyMotion_use_smartsign_us = 1
```

## 12、vim-repeat

重复操作插件，使用<kbd>.</kbd>即可重复操作

 github:https://github.com/tpope/vim-repeat

plugins.yaml

```yaml
- {repo: tpope/vim-repeat , on_map: .* }
```

## 13、vim-mundo

访问vim的undo树的插件

github:https://github.com/simnalamburt/vim-mundo

plugins.yaml

```yaml
- {repo: simnalamburt/vim-mundo , on_map: { n: <Plug> } }
```

按说这个插件，命令模式`:GundoToggle`能够唤起，但是不起作用。稍后调试一下

## 14、vim-quickrun

快速运行插件

github:https://github.com/thinca/vim-quickrun

plugins.yaml

```yaml
- repo: thinca/vim-quickrun
  on_cmd: QuickRun
  hook_add: |
        let g:quickrun_config = {
            \   "_" : {
            \       "outputter" : "message",
            \   },
            \}
        let g:quickrun_no_default_key_mappings = 1

```

## 15、dash.vim

这个插件是用在Mac上的，因为只有mac才有`Dash.app`,在其他操作系统上，将不起作用

github:https://github.com/rizzatti/dash.vim

plugins.yaml

```yaml
- repo: rizzatti/dash.vim
  on_map: { n: <Plug> }
  hook_add: |
      let g:dash_map = {
        \ 'javascript': ['javascript', 'NodeJS'],
        \ 'javascript.jsx': ['react'],
        \ 'html': ['html', 'svg'],
        \ 'go' : 'Go',
        \}

```

## 16、comfortable-motion.vim

光标位置不变的上下滚动

github:https://github.com/yuttie/comfortable-motion.vim

plugins.yaml

```yaml
- repo: yuttie/comfortable-motion.vim
  on_func: comfortable_motion#flick
  hook_add: |
        let g:comfortable_motion_no_default_key_mappings = 1
        let g:comfortable_motion_impulse_multiplier = 1

```

## 17、 vim-easy-align

快速格式化对其，例如编程语言中的赋值多行对其

github:https://github.com/junegunn/vim-easy-align

plugins.yaml

```yaml
- repo: junegunn/vim-easy-align
  on_ft: [vim,json,go,html,js,jsx,py,css,less,tmpl,toml,xml,sql,Dockerfile]

```

## 18、vim-startify

这个用来选择编辑过的文件菜单的插件

github:https://github.com/mhinz/vim-startify

plugins.yaml

```yaml
- repo: mhinz/vim-startify
  on_cmd: Startify
  depends: vim-devicons
  hook_source: source  $VIM_PATH/layers/+ui/startify/config.vim
  hook_post_source: |
      function! StartifyEntryFormat()
        return 'WebDevIconsGetFileTypeSymbol(absolute_path) ." ". entry_path'
      endfunction
```

$VIM_PATH/layers/+ui/startify/config.vim

```css

" For startify
let g:startify_padding_left = 30
let s:header = [
      \ '',
      \ '   __         _    _        _    _      _         _    ',
      \ '  / /    ___ | |_ ( ) ___  | |_ | |__  (_) _ __  | | __',
      \ ' / /    / _ \| __||/ / __| | __|| |_ \ | || |_ \ | |/ /',
      \ '/ /___ |  __/| |_    \__ \ | |_ | | | || || | | ||   < ',
      \ '\____/  \___| \__|   |___/  \__||_| |_||_||_| |_||_|\_\',
      \ '                                                       ',
      \ '             [ ThinkVim   Author:taigacute ]           ',
      \ '',
      \ ]

let s:footer = [
      \ '+-------------------------------------------+',
      \ '|            ThinkVim ^_^                   |',
      \ '|    Talk is cheap Show me the code         |',
      \ '|                                           |',
      \ '|            GitHub:taigacute               |',
      \ '+-------------------------------------------+',
      \ ]

function! s:center(lines) abort
  let longest_line   = max(map(copy(a:lines), 'strwidth(v:val)'))
  let centered_lines = map(copy(a:lines),
        \ 'repeat(" ", (&columns / 2) - (longest_line / 2)) . v:val')
  return centered_lines
endfunction

let g:startify_custom_header = s:center(s:header)
let g:startify_custom_footer = s:center(s:footer)

```

## 19、vim-choosewin

选择窗口插件，类似于tmux的`<C-b>q`

github:https://github.com/t9md/vim-choosewin

plugins.yaml

```css
- repo: t9md/vim-choosewin
  on_map: { n: <Plug> }
  hook_source: source  $VIM_PATH/layers/+tools/choosewin/config.vim  
```

$VIM_PATH/layers/+tools/choosewin/config.vim

```css

" Plugin: vim-choosewin
" ---------------------------------------------------------
let g:choosewin_label = 'SDFJKLZXCV'
let g:choosewin_overlay_enable = 1
let g:choosewin_statusline_replace = 1
let g:choosewin_overlay_clear_multibyte = 0
let g:choosewin_blink_on_land = 0

let g:choosewin_color_label = {
	\ 'cterm': [ 236, 2 ], 'gui': [ '#555555', '#000000' ] }
let g:choosewin_color_label_current = {
	\ 'cterm': [ 234, 220 ], 'gui': [ '#333333', '#000000' ] }
let g:choosewin_color_other = {
	\ 'cterm': [ 235, 235 ], 'gui': [ '#333333' ] }
let g:choosewin_color_overlay = {
	\ 'cterm': [ 2, 10 ], 'gui': [ '#88A2A4' ] }
let g:choosewin_color_overlay_current = {
	\ 'cterm': [ 72, 64 ], 'gui': [ '#7BB292' ] }
```

## 20、**accelerated-jk**

让`j``k`键，上下翻动时，增加速度，拥有滚动效果

github:<https://github.com/rhysd/accelerated-jk> 

plugins.yaml

```yaml
- { repo: rhysd/accelerated-jk, on_map: { n: <Plug> } }
```

## 21、vim-devicons 

增加`icon`图标的插件

github:<https://github.com/ryanoasis/vim-devicons> 

plugins.yaml

```yaml
# Interface{{{
- repo: ryanoasis/vim-devicons
  hook_add: let g:webdevicons_enable_denite = 1
```

## 22、vim-snippets

模板插件，增加写代码速度

github:<https://github.com/honza/vim-snippets> 

plugins.yaml

```yaml
- repo: honza/vim-snippets
  depends : coc.nvim
```

## 23、coc.nvim

一个提供插件的插件

github:https://github.com/neoclide/coc.nvim

plugins.yaml

```yaml
- repo: neoclide/coc.nvim
  merge: 0
  rev: release
  hook_add: source  $VIM_PATH/layers/+completion/coc/config.vim

```

- ` $VIM_PATH/layers/+completion/coc/config.vim`

  ```css
  "CoC configlet 
  let g:coc_snippet_next = '<TAB>'
  let g:coc_snippet_prev = '<S-TAB>'
  let g:coc_status_error_sign = '•'
  let g:coc_status_warning_sign = '•'
  let g:coc_global_extensions =['coc-html','coc-css','coc-snippets','coc-prettier','coc-eslint','coc-emmet','coc-tsserver','coc-pairs','coc-json','coc-python','coc-imselect','coc-highlight','coc-git','coc-emoji','coc-lists','coc-post','coc-stylelint','coc-yaml','coc-template','coc-tabnine','coc-marketplace','coc-gitignore','coc-yank','coc-explorer','coc-go']
  
  augroup MyAutoCmd
    autocmd!
    " Setup formatexpr specified filetype(s).
    autocmd FileType typescript,json setl formatexpr=CocAction('formatSelected')
    " Update signature help on jump placeholder
    autocmd User CocJumpPlaceholder call CocActionAsync('showSignatureHelp')
  augroup end
  
  " Highlight symbol under cursor on CursorHold
  autocmd CursorHold * silent call CocActionAsync('highlight')
  
  "Use tab for trigger completion with characters ahead and navigate
  inoremap <silent><expr> <TAB>
        \ pumvisible() ? "\<C-n>" :
        \ <SID>check_back_space() ? "\<TAB>" :
        \ coc#refresh()
  
  inoremap <expr><S-TAB> pumvisible() ? "\<C-p>" : "\<C-h>"
  "inoremap <expr> <cr> pumvisible() ? "\<C-y>" : "\<C-g>u\<CR>"
  inoremap <silent><expr> <cr> pumvisible() ? coc#_select_confirm() : "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"
  
  function! s:check_back_space() abort
    let col = col('.') - 1
    return !col || getline('.')[col - 1]  =~# '\s'
  endfunction
  
  
  ```

## 24、vim-buffet

快速切换`buffer`的插件

github:https://github.com/bagrat/vim-buffet

plugins.yaml

```yaml
- repo: bagrat/vim-buffet
  hook_add: source $VIM_PATH/layers/+ui/buffet/config.vim

```

- `$VIM_PATH/layers/+ui/buffet/config.vim`

  ```css
  
  let g:buffet_tab_icon = "\uf00a"
  function! g:BuffetSetCustomColors()
      hi! BuffetCurrentBuffer cterm=NONE ctermbg=106 ctermfg=8 guibg=#b8bb26 guifg=#000000
      hi! BuffetTrunc cterm=bold ctermbg=66 ctermfg=8 guibg=#458588 guifg=#000000
      hi! BuffetBuffer cterm=NONE ctermbg=239 ctermfg=8 guibg=#504945 guifg=#000000
      hi! BuffetTab cterm=NONE ctermbg=66 ctermfg=8 guibg=#458588 guifg=#000000
      hi! BuffetActiveBuffer cterm=NONE ctermbg=10 ctermfg=239 guibg=#999999 guifg=#504945
  endfunction
  
  ```

## 25、spaceline.vim

状态条插件

github:https://github.com/hardcoreplayers/spaceline.vim

plugins.yaml

```yaml
- repo: taigacute/spaceline.vim
```

## 26、dein.vim

插件管理

github:https://github.com/Shougo/dein.vim

plugins.yaml

```yaml
- repo: Shougo/dein.vim
```

## 27、dockerfile.vim

高亮dockerfiles语法插件

github:https://github.com/honza/dockerfile.vim

plugins.yaml

```yaml
- { repo: honza/dockerfile.vim, on_ft: Dockerfile }
```

## 28、vim-emoji

使vim支持emoji的插件

github:https://github.com/junegunn/vim-emoji

plugins.yaml

```yaml
- { repo: junegunn/vim-emoji, on_ft: [markdown,vim] }

```

## 29、typescript-vim

TypeScrpit语法高亮

github:https://github.com/leafgarland/typescript-vim

plugins.yaml

```yaml
- { repo: leafgarland/typescript-vim, on_ft: [typescript.tsx,typescript] }

```

## 29、peitalin/vim-jsx-typescript

Syntax highlighting for JSX in Typescript.

github:https://github.com/peitalin/vim-jsx-typescript

plugins.yaml

```yaml
- { repo: peitalin/vim-jsx-typescript, on_ft: [typescript.tsx]}

```

## 30、vim-python-pep8-indent

python的PEP8缩进

github:https://github.com/Vimjas/vim-python-pep8-indent

plugins.yaml

```yaml
- { repo: Vimjas/vim-python-pep8-indent, on_ft: python }

```

## 31、SimpylFold

python的正确折叠插件

github:https://github.com/tmhedberg/SimpylFold

plugins.yaml

```yaml
- { repo: tmhedberg/SimpylFold, on_ft: python }

```

## 32、**python_match.vim**

python`%`匹配跳转

github:https://github.com/vim-scripts/python_match.vim

plugins.yaml

```yaml
- { repo: vim-scripts/python_match.vim, on_ft: python }

```

## 33、vim-python/python-syntax

增强高亮python语法

github:https://github.com/vim-python/python-syntax

plugins.yaml

```yaml
- repo: vim-python/python-syntax
  on_ft: python
  hook_add: let g:python_highlight_all = 1

```

## 34、vim-jsx-improve

Makes your javascript files support React jsx correctly.

github:https://github.com/neoclide/vim-jsx-improve

plugins.yaml

```yaml
- { repo: neoclide/vim-jsx-improve, on_ft: [javascript,jsx,javascript.jsx]}

```

## 35、vim-toml

TOML语法高亮

github:https://github.com/cespare/vim-toml

plugins.yaml

```yaml
- { repo: cespare/vim-toml, on_ft: toml }

```

## 36、**xml.vim**

xml编辑插件，自动补齐

github:https://github.com/vim-scripts/xml.vim

plugins.yaml

```yaml
- { repo: vim-scripts/xml.vim, on_ft: xml}

```

## 37、**ansible-vim**

配置自动化脚本高亮插件

This is a vim syntax plugin for Ansible 2.x, it supports YAML playbooks, Jinja2 templates, and Ansible's `hosts` files.

github:https://github.com/pearofducks/ansible-vim

plugins.yaml

```yaml
- { repo: pearofducks/ansible-vim, on_ft: [ yaml.ansible, ansible_hosts ]}

```

## 38、**vim-json**

json高亮插件

github:https://github.com/elzr/vim-json

plugins.yaml

```yaml
- repo: elzr/vim-json
  on_ft: json
  hook_add: let g:vim_json_syntax_conceal = 0

```

## 39、vim-go

Go语言对Vim支持

github:https://github.com/fatih/vim-go

plugins.yaml

```yaml
- repo: fatih/vim-go
  on_ft: go
  hook_source: source  $VIM_PATH/layers/+lang/go/config.vim

```

- `$VIM_PATH/layers/+lang/go/config.vim`

  ```css
  "vim-go
  let g:go_fmt_command = "goimports"
  let g:go_highlight_types = 1
  let g:go_highlight_fields = 1
  let g:go_highlight_functions = 1
  let g:go_highlight_function_calls = 1
  let g:go_highlight_methods = 1
  let g:go_highlight_structs = 1
  let g:go_highlight_operators = 1
  let g:go_highlight_extra_types = 1
  let g:go_highlight_build_constraints = 1
  let g:go_highlight_generate_tags = 1
  "disable use K to run godoc
  let g:go_doc_keywordprg_enabled = 0
  let g:go_def_mapping_enabled = 0
  ```

## 40、vim-markdown

Vim对Markdown的支持

github:https://github.com/tpope/vim-markdown

plugins.yaml

```yaml
- repo: tpope/vim-markdown
  on_ft: markdown
  hook_add: |
        let g:markdown_fenced_languages = [
          \ 'html',
          \ 'bash=sh',
          \ 'css',
          \ 'javascript',
          \ 'js=javascript',
          \ 'go',
          \]

```

## 41、**caw.vim**

Vim注释插件

github:https://github.com/tyru/caw.vim

plugins.yaml

```yaml
- repo: tyru/caw.vim
  on_map: { nx: <Plug> }

```

## 42、neoformat

格式化插件

github:https://github.com/sbdchd/neoformat

plugins.yaml

```yaml
- repo: sbdchd/neoformat
  on_cmd: [Neoformat,Neoformat!]
  hook_source: source  $VIM_PATH/layers/+tools/neoformat/config.vim

```

- `$VIM_PATH/layers/+tools/neoformat/config.vim`

  ```css
  
  let g:neoformat_try_formatprg = 1
  let g:jsx_ext_required = 0
  let g:neoformat_enabled_javascript=['prettier']
  let g:neoformat_enabled_html=['js-beautify']
  
  ```

## 43、**indentLine**

缩进线

github:https://github.com/Yggdroot/indentLine

plugins.yaml

```yaml
- repo: Yggdroot/indentLine
  on_ft: [python,html,css,vim,javascript,jsx,javascript.jsx,vue]
  hook_source: source $VIM_PATH/layers/+ui/indentline/config.vim

```

- `$VIM_PATH/layers/+ui/indentline/config.vim`

  ```css
  
  let g:indentline_enabled = 1
  let g:indentline_char='┆'
  let g:indentLine_fileTypeExclude = ['defx', 'denite','startify','tagbar','vista_kind','vista']
  let g:indentLine_concealcursor = 'niv'
  let g:indentLine_color_term = 96
  let g:indentLine_color_gui= '#725972'
  let g:indentLine_showFirstIndentLevel =1
  
  ```

## 44、vista.vim

new tagbar

参考：https://www.zhihu.com/question/31934850

github:https://github.com/liuchengxu/vista.vim

plugins.yaml

```yaml
- repo: liuchengxu/vista.vim
  on_cmd: [Vista,Vista!,Vista!!]
  hook_source: source  $VIM_PATH/layers/+tools/vista/config.vim

```

- `$VIM_PATH/layers/+tools/vista/config.vim`

  ```yaml
  let g:vista#renderer#enable_icon = 1
  let g:vista_default_executive = 'ctags'
  let g:vista_fzf_preview = ['right:50%']
  
  let g:vista_executive_for = {
    \ 'go': 'ctags',
    \ 'javascript': 'coc',
    \ 'javascript.jsx': 'coc',
    \ 'python': 'ctags',
    \ }
  
  ```

## 45、Emmet-vim

快速生成`HTML`的插件

参考：https://www.jianshu.com/p/ad8a6a786054

github:https://github.com/mattn/emmet-vim

plugins.yaml

```css
- repo: mattn/emmet-vim
  on_ft: [html,css,jsx,javascript,javascript.jsx]
  on_event: InsertEnter
  hook_add: |
        let g:use_emmet_complete_tag = 0
        let g:user_emmet_install_global = 0
        let g:user_emmet_install_command = 0
        let g:user_emmet_mode = 'i'
        let g:user_emmet_leader_key='<C-g>'
        let g:user_emmet_settings = {
            \  'javascript.jsx' : {
            \      'extends' : 'jsx',
            \  },
        \}

```

## 46、**rainbow**

彩虹括号增强版，提高复杂代码的阅读性

github:https://github.com/luochen1990/rainbow

plugins.yaml

```yaml
- repo: luochen1990/rainbow
  on_ft: [python,javascript,jsx,javascript.jsx,html,css,go,vim,toml]
  hook_source: let g:rainbow_active = 1

```

## 47、echodoc.vim

显示函数文档到command line中

github:https://github.com/Shougo/echodoc.vim

plugins.yaml

```css
- repo: Shougo/echodoc.vim
  on_event: CompleteDone
  hook_source: |
        call echodoc#enable()
        let g:echodoc#type = "virtual"

```

## 48、markdown-preview.nvim

预览Markdown的插件

github:https://github.com/iamcco/markdown-preview.nvim

plugins.yam

```css
- repo: iamcco/markdown-preview.nvim
  on_ft: [markdown,pandoc.markdown,rmd]
  hook_post_source: 'call mkdp#util#install()'
  hook_source: |
      let g:mkdp_auto_start = 1

```

## 49、goyo.vim

清爽模式，不被其他元素打扰的写作插件

参考：https://github.com/junegunn/goyo.vim

plugins.yaml

```css
- repo: junegunn/goyo.vim
  on_cmd: Goyo
  hook_source: source  $VIM_PATH/layers/+tools/goyo/config.vim

```

- `$VIM_PATH/layers/+tools/goyo/config.vim`

  ```css
  " Goyo
  
  " s:goyo_enter() "{{{
  " Disable visual candy in Goyo mode
  function! s:goyo_enter()
  	if has('gui_running')
  		" Gui fullscreen
  		set fullscreen
  		set background=light
  		set linespace=7
  	elseif exists('$TMUX')
  		" Hide tmux status
  		silent !tmux set status off
  	endif
  
  	" Activate Limelight
  	Limelight
  endfunction
  
  " }}}
  " s:goyo_leave() "{{{
  " Enable visuals when leaving Goyo mode
  function! s:goyo_leave()
  	if has('gui_running')
  		" Gui exit fullscreen
  		set nofullscreen
  		set background=dark
  		set linespace=0
  	elseif exists('$TMUX')
  		" Show tmux status
  		silent !tmux set status on
  	endif
  
  	" De-activate Limelight
  	Limelight!
  endfunction
  " }}}
  
  " Goyo Commands {{{
  autocmd! User GoyoEnter
  autocmd! User GoyoLeave
  autocmd  User GoyoEnter nested call <SID>goyo_enter()
  autocmd  User GoyoLeave nested call <SID>goyo_leave()
  " }}}
  
  " vim: set foldmethod=marker ts=2 sw=2 tw=80 noet :
  
  ```

## 50、limelight.vim

在使用goyo.vim的清爽模式时，块高亮插件

github:https://github.com/junegunn/Limelight.vim

plugins.yaml

```css
- repo: junegunn/Limelight.vim
  on_cmd: Limelight

```

## 51、splitjoin.vim

多行和单行之间快速切换的插件

github:https://github.com/AndrewRadev/splitjoin.vim

plugins.yaml

```yaml
- { repo: AndrewRadev/splitjoin.vim, on_map: { n: <Plug>Splitjoin }}

```

## 52、vim-expand-region

扩展选择插件，让你快速选择

github:https://github.com/terryma/vim-expand-region

plugins.yaml

```yaml
- { repo: terryma/vim-expand-region, on_map: { x: <Plug> }}

```

## 53、vim-textobj-user

用户可以自定文本块的插件

github:https://github.com/kana/vim-textobj-user

plugins.yaml

```yaml
- { repo: kana/vim-textobj-user, on_func: textobj#user# }

```

## 54、vim-operator-user

用户定义自己的操作符的插件

github:https://github.com/kana/vim-operator-user

plugins.yaml

```yaml
- { repo: kana/vim-operator-user, lazy: 1 }

```

## 55、vim-niceblock

更好的块选择插件

github:https://github.com/kana/vim-niceblock

plugins.yaml

```yaml
- { repo: kana/vim-niceblock, on_map: { x: <Plug> }}

```

## 56、**vim-smartchr**

一键插入多个候选的插件

github:https://github.com/kana/vim-smartchr

plugins.yaml

```yaml
- repo: kana/vim-smartchr
  on_event: InsertCharPre

```

## 57、vim-operator-replace

操作符替换插件

github:https://github.com/kana/vim-operator-replace

plugins.yaml

```yaml
- repo: kana/vim-operator-replace
  depends: vim-operator-user
  on_map: { vnx: <Plug> }

```

## 58、vim-sandwich

改变括号的插件

github:https://github.com/machakann/vim-sandwich

plugins.yaml

```yaml
- repo: machakann/vim-sandwich
  on_map: { vnx: [<Plug>(operator-sandwich-,<Plug>(textobj-sandwich-]}

```

## 59、vim-textobj-multiblock

过个括号一起使用的文本快

github:https://github.com/osyo-manga/vim-textobj-multiblock/

plugins.yaml

```yaml
- repo: osyo-manga/vim-textobj-multiblock
  depends: vim-textobj-user
  on_map: { ox: <Plug> }
  hook_add:  let g:textobj_multiblock_no_default_key_mappings = 1

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

## 4、easymotion

| 快捷键                                            | 功能                   |
| ------------------------------------------------- | ---------------------- |
| <kbd>LEADER</kbd><kbd>LEADER</kbd> + <kbd>b</kbd> | 上一个单词的快速移动   |
| <kbd>LEADER</kbd><kbd>LEADER</kbd> + <kbd>w</kbd> | 下一个单词的快速移动   |
| <kbd>LEADER</kbd><kbd>LEADER</kbd> + <kbd>f</kbd> | 查找一个字母的快速移动 |

## 5、quickrun

<kbd>LOCALLEADER</kbd> + <kbd>r</kbd>

或:`:QuickRun`

## 6、comfortable-motion.vim

| 快捷键                         | 功能                     |
| ------------------------------ | ------------------------ |
| <kbd>Ctrl</kbd> + <kbd>d</kbd> | 光标所在行不变，向下滚动 |
| <kbd>Ctrl</kbd> + <kbd>u</kbd> | 光标位置不变，向上滚动   |
|                                |                          |

## 7、vim-easy-align

引导键:<kbd>g</kbd><kbd>a</kbd>

常用快捷键：

- `vipga=`
  - `v`isual-select `i`nner `p`aragraph
  - Start EasyAlign command (`ga`)
  - Align around `=`
- `gaip=`
  - Start EasyAlign command (`ga`) for `i`nner `p`aragraph
  - Align around `=`

## 8、vim-startify

打开：<kbd>LEADER</kbd> + <kbd>s</kbd>

## 9、vim-choosewin

打开：<kbd>LEADER</kbd> + <kbd>-</kbd>



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