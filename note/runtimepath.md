公司的 10.61.33.10 自从上次重装系统后，vim 就一直无法语法高亮，用得很不爽。经过网上大搜索和实验研究，发现启动 vim 后依次输入以下命令 

`:let $VIMRUNTIME = "/usr/share/vim/vim61"`

` :set runtimepath=/usr/share/vim/vim61` 

`:syntax on` 

就 OK 了。原因是默认的环境变量对应的路径是错误的，路径中多了一层“local”: `$VIMRUNTIME="/usr/local/share/vim"`，可用 `:echo $VIMRUNTIME` 命令查看。 `runtimepath=~/.vim,/usr/local/share/vim/vimfiles,/usr/local/share/vim, /usr/local/share/vim/vimfiles/after,~/.vim/after`，可用 `:set rtp` 或 `:set runtimepath` 命令查看。 但每次进入 vim 都要输入这几条命令，相当麻烦，解决方法是把这三条命令写入 /root/.vimrc，每次 vim 启动时都会先执行这里面的脚本。相比之下，10.61.102.11 的语法高亮可用，因为它的环境变量的路径已经设对了，但它在 /root/ 下并没有 .vimrc 这个文件。这么看来，两台机必然至少有一个保存着这些环境变量文件有所不同，是哪些文件呢？ 在 10.61.33.10 下输入命令 `grep -r "/usr/local/share/vim"` /etc/ 搜不到任何结果。 grep -r "/usr/local/share/vim" / 搜了几个小时还没有任何结果，放弃了。  看来是以非明文的方式放在某个文件里，也就是说是编译 Linux 内核或安装 vim 时决定的。 