# Dictionaries

Dictionaries I managed to collect and generate using a python script that I wrote <https://github.com/nl253/DictGen>

-   frequent.dict

      Generated using my DictGen script from analysing word frequencies of:

            + All Harry Potter books (for regular english words)
            + A few man pages
            + A few computer science textbooks

      The words have been 'normalised', numbers removed, case removed, very, very
      high quality.

-   css.dict

      This one is very good as well.

-   erlang.dict

-   haskell.dict

-   java.dict

-   javascript.dict

-   perl.dict

-   php.dict

-   python.dict

    > This one I got from someone on GitHub but tbh it is a little bit too
    > `complete`.  2.4 MB is an overkill and you will see a lot of redundant
    > sections from external libraries, frameworks you might not use ... 
    > My advice, stick to omnicompletion from jedi or rope.
    > I recommend : https://github.com/python-mode/python-mode

-   shell.dict (tiny)

-   sql.dict

    > Also generated by me, it looks decent, it only includes upper case entries.
    > Because sql is not a particularly larg language it only includes about 400
    > entries.

-   thesaurus.txt

      An amazing thesaurus, highly recommended.

-   en.utf-8.add

      Extra spelling entries from vim

-------------------------------------------------------------------------

## How do I use it in (N)Vim ?

NOTE before you run any scripts always verify them.

> BASIC SETUP 

Download with curl and put into ~/.dicts or wherever you like.

```sh
mkdir -p ~/.dicts
curl -fLo ~/.dicts/thesaurus.txt https://raw.githubusercontent.com/nl253/Dictionaries/master/thesaurus.txt
curl -fLo ~/.dicts/frequent.dict https://raw.githubusercontent.com/nl253/Dictionaries/master/frequent.dict
```

If you are a vim user and don't know how to add them as a dict / thesaurus
This will append it to the end of file.

For Nvim : 

```sh
echo "set dictionary=~/.dicts/frequent.dict" >> "${HOME}/.config/nvim/init.vim"
echo "set thesaurus=~/.dicts/thesaurus.txt" >> "${HOME}/.config/nvim/init.vim"
```
For Vim : 

```sh
echo "set dictionary=~/.dicts/frequent.dict" >> "${HOME}/.vimrc"
echo "set thesaurus=~/.dicts/thesaurus.txt" >> "${HOME}/.vimrc"
```

> ADVANCED (better if you have a lot of configuration for a lot of filetypes)
> If you are using vim or neovim I recommend this trick : 
> (It will give you very good plugin-free completion.)

You will need
- **curl**
- to **change variable names** at the beginning

```vim
" make sure these are declared before you use them, ideally at the very beignning
" set a dir that suits you to store all dicts
let g:DICT_DIR = glob('~/.dicts/')

" define PROGRAMMING filetypes that you work with 
let g:PROGRAMMING =  [ 'vim', 'xhtml', 'html', 'css',
            \'javascript', 'python', 'php', 'sh', 'zsh' ]

" define MARKUP filetypes that you work with 
let g:MARKUP = [ 'markdown', 'rst' ]

" no need to tweak this 
let g:MARKUP_EXT = ['md', 'rst', 'org', 'txt', 'textile'] 

if empty(g:DICT_DIR)
    " make the dict dir if it doesn't exits
    call system('mkdir -p ' . g:DICT_DIR)
endif

" THESE NEED!!! TO BE RELATIVE TO $HOME
"
" Scripts == ~/Scripts
" .templates == ~/.templates
" Ruby/Something/Alice == ~/Ruby/Something/Alice
" and so on

" point vim to dirs where you have your projects
"
let g:WORKING_DIRS = [ 'Scripts', 'Notes','.scratchpads', 
            \'.templates', 'Projects', '.bin', '.dicts']

"
" download dictionaries from GitHub {{{
"
" choose those you use from :
" ---------------------------
" css.dict erlang.dict frequent.dict
" haskell.dict java.dict javascript.dict 
" perl.dict php.dict python.dict
" sh.dict sql.dict thesaurus.txt
" mysql.txt often-misspelled.dict
"
let g:DICTS = ['frequent.dict', 'thesaurus.txt', 'css.dict', 'sql.dict', 'sh.dict', 'javascript.dict']

 "UNCOMMENT IN NEED
" let g:DICTS += ['erlang.dict', 'php.dict', 'haskell.dict', 'perl.dict', 'java.dict'] 

for dict in g:DICTS
    if ! filereadable(g:DICT_DIR . dict)
        execute '!curl -fLo ' . g:DICT_DIR . dict . ' https://raw.githubusercontent.com/nl253/Dictionaries/master/' . dict
    endif
endfor
" }}}

" general english dict with thesaurus set
if filereadable(g:DICT_DIR . 'thesaurus.txt')
    execute 'set thesaurus=' . g:DICT_DIR . 'thesaurus.txt'
endif

if filereadable(g:DICT_DIR . 'frequent.dict')
    execute 'set dictionary=' .  g:DICT_DIR . 'frequent.dict'
endif

" call when filetype is set 
function! Init()
    if index(g:MARKUP, &filetype) >= 0  " this will be execute for g:MARKUP filetypes 
        setlocal complete=.,w,k,s  " for markup  :: current buffer, windows (splits), dict, thesaurus
        for dir in g:WORKING_DIRS  " this actually isn't recursive 
            for extension in g:MARKUP_EXT " 1 levels of depth by default
                execute 'setl complete+=k~/'.dir.'/**.'.extension
                " uncomment to get 2 levels of depth
                " execute 'setl complete+=k~/'.dir.'/**/**.'.extension  
                " uncomment to get 3 levels of depth
                " execute 'setl complete+=k~/'.dir.'/**/**/**.'.extension  
            endfor
        endfor
    " this will be execute for g:PROGRAMMING filetypes
    elseif index(g:PROGRAMMING, &filetype) >= 0                             
        if expand('%:e') != ''     " if there is an extension (needed)
            setl complete=.,w,t    " current buffer, windows (splits), tags
            for dir in g:WORKING_DIRS
                execute 'setl complete+=k~/'.dir.'/**.'.expand('%:e')
                " uncomment to get 2 levels of depth
                "execute 'setl complete+=k~/'.dir.'/**/**.'.expand('%:e')
                " uncomment to get 3 levels of depth
                "execute 'setl complete+=k~/'.dir.'/**/**/**.'.expand('%:e')
            endfor
        endif
        if filereadable(g:DICT_DIR . &filetype . '.dict')                       
            " if in g:PROGRAMMING and an appropriate dict is available, 
            " then REPLACE english dict with it and add dict to user defined completion 
            " this works because dicts follow the naming convention of {filetype}.dict
            let g:to_exe = 'setl dictionary='. g:DICT_DIR . &filetype . '.dict' " 
            execute g:to_exe                                                    
            setl complete+=k
        endif
    endif
endfunction

au! FileType * call Init()
```

NOTE depth by default for this is 1 level. You might want more.

PLEASE CHANGE VARIABLES:

    - g:WORKING_DIRS (list of dirs relative to ~ )
    - g:DICT_DIR     (string)
    - g:MARKUP       (list of markup filetypes)
    - g:MARKUP_EXT   (list of markup extensions you want to collect words from)
    - g:PROGRAMMING  (list of programming filetypes)
    - g:DICTS        (list of dicts you want to download)
    
### What it does 

#### For g:PROGRAMMING types:
1. look in all g:WORKING_DIRS for files with the same extension 
2. add them to completion 

Also:
- if you have a dict for this programming language (eg for java java.dict),
  replace the english dict (here frequent.dict) with this dict and add that
  dict to user completion

#### For g:MARKUP types:
1. look in all g:WORKING_DIRS for files extensions specified by g:MARKUP_EXT.
2. add to user defined completion 

----------------------------------------------------------------------------------------------

Now for markup languages you will get completion for things you have written before, like places,
words you made up, and your name, address etc. 

For programming languages you will get completion for relevant variable names and things you've written before.

Now when you press CTRL-N (or CTRL-P)

![Alt text](screenshot.png?raw=true "popupmenu") 
![Alt text](screenshot2.png?raw=true "popupmenu") 
![Alt text](screenshot3.png?raw=true "popupmenu") 


If there are any issues please let me know.
