---
title: "Windows-Unity3D-Vim"
date: 2023-09-02
categories: ["Development", "Unity3D"]
tags: ["unity3d", "vim", "windows"]
---
## 기본 참고
https://forum.unity.com/threads/unity3d-with-vim.355814/

## 문제 상황
### 파이썬 라이브러리 로딩 문제
"Sorry, this command is disabled, the Python library could not be loaded."

YouCompleteMe를 설치하려니까 발생한 문제

##### 해결책 : https://vi.stackexchange.com/a/38332

기본적으로 Vim이 사용하기로 계획한 Python의 버전이 맞지 않아서 생긴 문제
위에서는 뭐 자세하게 항목을 나열했는데 간단하게 해결하는 방법은 Vim Release Github에서 Interface Information이라 나열된 항목의 링크에서 다운받으면 해결 가능

![](/images/19d834e5-c1d3-4d6f-af8a-18114758e184-image.PNG)

### omnisharp 설치 문제
"fialed to install omnisharp-roslyn server"
PowerShell의 권한으로 발생한 문제로 보임
PowerShell Script인 ps1을 수행할 수 없어서 발생한 문제로 짐작됨

일단 PowerShell의 권한을 내린 후
##### Reference : https://betterinvesting.tistory.com/276

vimfiles아래에 설치되는 omnisharp의 log를 보면 install.log파일 존재
해당 파일 1번 줄에 적힌 명령어를 관리자 권한으로 실행한 PowerShell에서 수동으로 입력하여 설치 진행

작업 후 PowerShell Script권한 원복

## Vim Plugin
```
set hlsearch " 검색어 하이라이팅
set nu " 줄번호
set autoindent " 자동 들여쓰기
set scrolloff=2
set wildmode=longest,list
set ts=4 "tag select
set sts=4 "st select
set sw=1 " 스크롤바 너비
set autowrite " 다른 파일로 넘어갈 때 자동 저장
set autoread " 작업 중인 파일 외부에서 변경됬을 경우 자동으로 불러옴
set cindent " C언어 자동 들여쓰기
set bs=eol,start,indent
set history=256
set laststatus=2 " 상태바 표시 항상
"set paste " 붙여넣기 계단현상 없애기
set shiftwidth=4 " 자동 들여쓰기 너비 설정
set showmatch " 일치하는 괄호 하이라이팅
set smartcase " 검색시 대소문자 구별
set smarttab
set smartindent
set softtabstop=4
set tabstop=4
set ruler " 현재 커서 위치 표시
set incsearch
set statusline=\ %<%l:%v\ [%P]%=%a\ %h%m%r\ %F\
set encoding=utf-8

" Mapping <leader> => ,
"let mapleader=","

" How many tenths of a second to blink when matching brackets
set mat=2

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"   Search Setting
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

" Ignore case when searching
set ignorecase

" Move cursor when searching
set incsearch

" Using tab like 4 space
"set expandtab
set smarttab

" Smart Indent
set si

""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""
"   Moving tab Setting
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

map <C-j> <C-W>j
map <C-k> <C-W>k
map <C-h> <C-W>h
map <C-l> <C-W>l

" 마지막으로 수정된 곳에 커서를 위치함
au BufReadPost *
\ if line("'\"") > 0 && line("'\"") <= line("$") |
\ exe "norm g`\"" |
\ endif

" 파일 인코딩을 한국어로
"if $LANG[0]=='k' && $LANG[1]=='o'
"set fileencoding=korea
"endif

" 구문 강조 사용
if has("syntax")
 syntax on
endif

" 컬러 스킴 사용
"colorscheme jellybeans
colorscheme onehalfdark
let g:airline_theme='onehalfdark'

filetype off
set shellslash

set rtp+=~/vimfiles/bundle/Vundle.vim
call vundle#begin('~/vimfiles/bundle')

Plugin 'VundleVim/Vundle.vim'
Plugin 'scrooloose/nerdtree'
Plugin 'https://github.com/jiangmiao/auto-pairs.git'
Plugin 'https://github.com/OmniSharp/omnisharp-vim.git'
Plugin 'https://github.com/kien/rainbow_parentheses.vim.git'
Plugin 'https://github.com/vim-syntastic/syntastic.git'
Plugin 'https://github.com/preservim/tagbar.git'
Plugin 'https://github.com/vim-airline/vim-airline.git'
Plugin 'https://github.com/rafi/awesome-vim-colorschemes.git'
Plugin 'https://github.com/altercation/vim-colors-solarized.git'
Plugin 'https://github.com/tpope/vim-dispatch.git'
Plugin 'https://github.com/easymotion/vim-easymotion.git'
Plugin 'https://github.com/ycm-core/YouCompleteMe.git'
Plugin 'https://github.com/tpope/vim-fugitive.git'
Plugin 'https://github.com/kien/ctrlp.vim.git'
Plugin 'https://github.com/nanotech/jellybeans.vim.git'
Plugin 'https://github.com/sonph/onehalf.git'
Plugin 'https://github.com/jeetsukumaran/vim-buffergator.git'

call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
"

map <C-n> :NERDTreeToggle<CR>

"-----------------------------------------------------------------------"
" Aiarline
"-----------------------------------------------------------------------"
set laststatus=2
let g:airline#extensions#tabline#enabled = 1 "버퍼 목록 켜기
let g:airline#extensions#tabline#left_sep = ' '
let g:airline#extensions#tabline#left_alt_sep = '|'
" 파일명만 출력
let g:airline#extensions#tabline#fnamemod = ':t'
let g:airline_highlighting_cache = 1

let g:airline_powerline_fonts = 1
let g:airline_theme= 'minimalist'
let g:airline_section_y = '' 
let g:airline_section_warning= '' "마지막 status창 사용 안함
" 버퍼 목록 켜기
" 이 옵션은 버퍼를 수정한 직후 버퍼를 감춰지도록 한다.
" 이 방법으로 버퍼를 사용하려면 거의 필수다.
set hidden

" 버퍼 새로 열기
" 원래 이 단축키로 바인딩해 두었던 :tabnew를 대체한다.
"nmap <leader>T :enew<cr>

" 다음 버퍼로 이동
nmap <C-tab> :bnext<CR>

" 이전 버퍼로 이동
nmap <C-S-tab>h :bprevious<CR>

" 현재 버퍼를 닫고 이전 버퍼로 이동
" 탭 닫기 단축키를 대체한다.
"nmap <leader>bq :bp <BAR> bd #<CR>

" 모든 버퍼와 각 버퍼 상태 출력
"nmap <leader>bl :ls<CR>

"----------------------------------------------------------------------"
" ctrlp.vim 설정(파일 탐색 속도 향상)
"----------------------------------------------------------------------"
set wildignore+=*/tmp/*,*.so,*.a,*.swp,*.zip,*.obj  " MacOSX/Linux
set wildignore+=*\\tmp\\*,*.swp,*.zip,*.exe  		" Windows

let g:ctrlp_custom_ignore = '\v[\/]\.(git|hg|svn)$'
let g:ctrlp_custom_ignore = {
  \ 'dir':  '\.git$\|public$\|log$\|tmp$\|vendor$',
  \ 'file': '\v\.(exe|so|dll|a)$',
  \ 'link': 'some_bad_symbolic_links'
\ }
let g:ctrlp_max_files = 10000
let g:ctrlp_max_depth = 30
let g:ctrlp_follow_symlinks = 1
"let g:ctrlp_user_command = ['.git', 'cd %s && git ls-files -co --exclude-standard']
"let g:ctrlp_use_readdir = 0
let g:ctrlp_root_markers = ['ctrlp-marker']
let g:ctrlp_show_hidden = 1

" <c-f>, <c-b> 모드 변환(MRU(Most Recently Used)내 검색, file 전체 검색, buffers내 검색)
" <c-d> path내 검색어가 포함되어 검색 또는 오직 파일내 검색어만 포함 검색 
" <c-r> 정규 표현식 모드 전환(검색어와 완전히 일치한 파일만 보여 줌)
" <c-j>, <c-k> 검색결과내 커서 위아래 이동
" <c-t>, <c-v>, <c-x> 파일을 새로운 tab으로 열거나 창을 분활하여 파일을 염.
" <c-n>, <n-p> 검색 history의 next/previous 문자열 선택
" <c-z>, <c-o> 검색된 결과물에 <c-z>로 마크를 하고 <c-o>로 오픈(멀티마크 가능) 
" <c-y> 검색입력어로 된 파일을 만든다. 파일 위치는 검색 폴더의 최상위 위치
" :help ctrlp-mappings 키 맵핑에 관한 설명

" 단축키를 리더 키로 대체
nmap <C-p> :CtrlP<cr>

" 여러 모드를 위한 단축키
"nmap <leader>bb :CtrlPBuffer<cr>
"nmap <leader>bm :CtrlPMixed<cr>
"nmap <leader>bs :CtrlPMRU<cr>

" 화면 오른쪽을 사용
let g:buffergator_viewport_split_policy = 'R'

" 단축키를 직접 지정하겠음
let g:buffergator_suppress_keymaps = 1

" 버퍼 돌기 (Looper buffers)
"let g:buffergator_mru_cycle_loop = 1

" 이전 버퍼로 이동
"nmap <leader>jj :BuffergatorMruCyclePrev<cr>

" 다음 버퍼로 이동
"nmap <leader>kk :BuffergatorMruCycleNext<cr>

" 모든 버퍼 보기
nmap <C-b> :BuffergatorOpen<cr>

" 위의 첫번재 해결책과 공유하는 단축키 (버퍼 닫기를 뜻함)
"nmap <leader>T :enew<cr>
"nmap <leader>bq :bp <BAR> bd #<cr>
```

이번에 설치한 Plugin 목록
##### CtrlP와 BufferGator를 설치한 이유 : https://bakyeono.net/post/2015-08-13-vim-tab-madness-translate.html

그 외에 Path설정 문제라던가 32-64bit 일치 문제, msvc로 YouCompleteMe build, Vim-Plugin-Vundle같은건 패스