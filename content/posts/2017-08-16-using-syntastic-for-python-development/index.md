---
title: "Using Syntastic for Python development"
date: 2017-08-16
categories: 
  - "tech"
---

I use [Synstastic plugin](https://github.com/vim-syntastic/syntastic) of vim for syntax checking in vim. Syntastic offers syntax checking for a LOT of languages. But, there is a problem that i had been facing with it. For a file with larger than 4k lines, it takes a lot of time to check the syntax and it used to happen every time you save the file. Syntax checking on write operation is the default behavior.

So, i did some changes in my .vimrc so that i could still use Syntastic for larger files. Do note that syntastic checking still takes a long time but, i have configured it to be called whenever i want to rather than on every write operation or opening of file.

_" show list of errors and warnings on the current file_ _nmap <leader>e :Errors<CR>_ _" Whether to perform syntastic checking on opening of file_ _" This made it very slow on open, so don't_ _let g:syntastic\_check\_on\_open = 0_ _" Don't check every time i save the file_ _" I will call you when i need you_ _let g:syntastic\_check\_on\_wq = 0_ _" By default, keep syntastic in passive mode_ _let g:syntastic\_mode\_map = { 'mode': 'passive' }_ _" Use :Sc to perform syntastic check_ _:command Sc :SyntasticCheck_ _" Check pylint for python_ _let g:syntastic\_python\_checkers = \['pylint'\]_ _" For jsx - React and React native_ _let g:syntastic\_javascript\_checkers = \['eslint'\]_

This change made opening of a larger python file ~25s (yes, seconds) faster. It still takes a lot of time for syntax checking though. I will have to find out why and if i could do anything about it. I don't want to leave out this plugin because it offers so much. I could simply use [Python-mode](https://github.com/python-mode/python-mode) for python syntax checking but, what about the rest of the languages which i am going to use.
