julia-cell
============

Seamlessly run Python code from Vim in Julia, including executing individual
code cells similar to Jupyter notebooks and MATLAB.

julia-cell is especially suited for data exploration and visualization using
Python. You can for example define a code cell that loads your input data, and
another code cell to visualize the data. This plugin allows you to change and
re-run the visualization part of your code without having to reload the data
each time.


Demo
----

![Demo animation](../assets/julia-cell-demo.gif?raw=true)


Requirements
------------

julia-cell requires Vim or Neovim to be compiled with Python 2 or Python 3
support (`+python` or `+python3` when running `vim --version`). If both Python
versions are found, the plugin will prefer Python 3.

julia-cell depends on [vim-slime] to send the code to Julia, see
[Installation](#installation) instructions below.

Additionally, the cell execution feature requires Tkinter and a
[clipboard program](#supported-clipboard-programs) to be installed.
On Linux, the plugin supports [xclip] and [xsel] (preferring the former).
On macOS, the plugin will use pbcopy.
Windows systems are currently not supported.
There is also a verbose version of the cell execution feature that does not
require Tkinter or a clipboard program, see [Usage](#usage).

[xclip]: https://github.com/astrand/xclip
[xsel]: https://github.com/kfish/xsel
[vim-slime]: https://github.com/jpalardy/vim-slime


Installation
------------

It is easiest to install julia-cell using a plugin manager (I personally
recommend [vim-plug]). See respective plugin manager's documentation for more
information about how to install plugins.


### [vim-plug]

~~~vim
Plug 'jpalardy/vim-slime', { 'for': 'python' }
Plug 'hanschen/vim-julia-cell', { 'for': 'python' }
~~~


### [Vundle]

~~~vim
Plugin 'jpalardy/vim-slime'
Plugin 'hanschen/vim-julia-cell'
~~~


### [NeoBundle]

~~~vim
NeoBundle 'jpalardy/vim-slime', { 'on_ft': 'python' }
NeoBundle 'hanschen/vim-julia-cell', { 'on_ft': 'python' }
~~~


### [Dein]

~~~vim
call dein#add('jpalardy/vim-slime', { 'on_ft': 'python' })
call dein#add('hanschen/vim-julia-cell', { 'on_ft': 'python' })
~~~


### [Pathogen]

~~~sh
cd ~/.vim/bundle
git clone https://github.com/hanschen/vim-julia-cell.git
~~~

[vim-plug]: https://github.com/junegunn/vim-plug
[Vundle]: https://github.com/VundleVim/Vundle.vim
[NeoBundle]: https://github.com/Shougo/neobundle.vim
[Dein]: https://github.com/Shougo/dein.vim
[Pathogen]: https://github.com/tpope/vim-pathogen


Usage
-----

julia-cell sends code from Vim to Julia using [vim-slime]. For this to
work, Julia has to be running in a terminal multiplexer like GNU Screen or
tmux, or in a Vim or Neovim terminal. I personally use tmux, but you will find
`screen` installed on most *nix systems.

It is recommended that you familiarize yourself with [vim-slime] first before
using julia-cell. Once you understand vim-slime, using julia-cell will be a
breeze.

julia-cell does not define any key mappings by default, but comes with the
commands listed below, which I recommend that you bind to key combinations of
your likings. The [Example Vim Configuration](#example-vim-configuration) shows
some examples of how this can be done.

Note that the cell execution feature copies your code to the system clipboard.
You may want to avoid using this feature if your code contains sensitive data.


### Commands

    :JuliaCellExecuteCell

Execute the current code cell in Julia.
Requires a [clipboard program](#supported-clipboard-programs)

    :JuliaCellExecuteCellJump

Execute the current code cell in Julia, and jump to the next cell.
Requires a [clipboard program](#supported-clipboard-programs)

    :JuliaCellExecuteCellVerbose

Print and execute the current code cell in Julia.
Verbose version of `JuliaCellExecuteCell` that works without a clipboard
program.

    :JuliaCellExecuteCellVerboseJump

Print and execute the current code cell in Julia, and jump to the next cell.
Verbose version of `JuliaCellExecuteCellJump` that works without a clipboard
program.

    :JuliaCellRun

Run the whole script in Julia.

    :JuliaCellClear

Clear Julia screen.

    :JuliaCellClose

Close all figure windows.

    :JuliaCellPrevCell

Jump to the previous cell header.

    :JuliaCellNextCell

Jump to the next cell header.

    :JuliaCellPrevCommand

Run previous command.

    :JuliaCellRestart

Restart Julia.

[vim-slime]: https://github.com/jpalardy/vim-slime


Defining code cells
-------------------

Code cells are defined by either Vim marks or special text in the code,
depending on if `g:julia_cell_delimit_cells_by` is set to `'marks'` or
`'tags'`, respectively. The default is to use Vim marks (see `:help mark` in
Vim).

The examples below show how code cell boundaries work.


### Code cells defined using marks

Marks are depicted as letters in the left-most column.

~~~
                                   _
  | import numpy as np              | cell 1
  |                                _| 
a | numbers = np.arange(10)         | cell 2
  |                                 |
  |                                _|
b | for n in numbers:               | cell 3
  |     print(n)                   _|
c |     if n % 2 == 0:              | cell 4
  |         print("Even")           |
  |     else:                       |
  |         print("Odd")            |
  |                                _|
d | total = numbers.sum()           | cell 5
  | print("Sum: {}".format(total)) _|

~~~

Note that code cells can be defined inside statements such as `for` loops.
Julia's `%paste` will automatically dedent the code before execution.
However, if the code cell is defined inside e.g. a `for` loop, the code cell
*will not* iterate over the loop.

In the example above, executing cell 4 after cell 3 will only print `Odd` once
because Julia will execute the following code:

~~~python
for n in numbers:
    print(n)

~~~

for cell 3, followed by

~~~python
if n % 2 == 0:
    print("Even")
else:
    print("Odd")

~~~

for cell 4. The `for` statement is no longer included for cell 4.

You must therefore be careful when defining code cells inside statements.


### Code cells defined using tags

Using `##` to define cell boundaries.

~~~
                                   _
import numpy as np                  | cell 1
                                   _|
## Setup                            | cell 2
                                    |
numbers = np.arange(10)             |
                                   _|
## Print numbers                    | cell 3
                                    |
for n in numbers:                   |
    print(n)                        |
                                   _|
    ## Odd or even                  | cell 4
                                    |
    if n % 2 == 0:                  |
        print("Even")               |
    else:                           |
        print("Odd")                |
                                   _|
## Print sum                        | cell 5
                                    |
total = numbers.sum()               |
print("Sum: {}".format(total))      |
print("Done.")                     _|

~~~

See note in the previous section about defining code cells inside statements
(such as cell 4 inside the `for` loop in the example above).


Configuration
-------------

    g:julia_cell_delimit_cells_by

Specify if cells should be delimited by `'marks'` or `'tags'`.
Default: `'marks'`

    g:julia_cell_tag

If cells are delimited by tags, specify the format of the tags.
Default: `'##'`

    g:julia_cell_valid_marks

If cells are delimited by marks, specify which marks to use.
Default: `'abcdefghijklmnopqrstuvqxyzABCDEFGHIJKLMNOPQRSTUVWXYZ'`


Example Vim configuration
-------------------------

Here's an example of how to configure your `.vimrc` to use this plugin. Adapt
it to suit your needs.

~~~vim
" Load plugins using vim-plug
call plug#begin('~/.vim/plugged')
Plug 'jpalardy/vim-slime', { 'for': 'python' }
Plug 'hanschen/vim-julia-cell', { 'for': 'python' }
call plug#end()

"------------------------------------------------------------------------------
" slime configuration 
"------------------------------------------------------------------------------
" always use tmux
let g:slime_target = 'tmux'

" fix paste issues in ipython
let g:slime_python_ipython = 1

" always send text to the top-right pane in the current tmux tab without asking
let g:slime_default_config = {
            \ 'socket_name': get(split($TMUX, ','), 0),
            \ 'target_pane': '{top-right}' }
let g:slime_dont_ask_default = 1

"------------------------------------------------------------------------------
" julia-cell configuration
"------------------------------------------------------------------------------
" Use '##' to define cells instead of using marks
let g:julia_cell_delimit_cells_by = 'tags'

" Keyboard mappings. <Leader> is \ (backslash) by default

" map <Leader>r to run script
nnoremap <Leader>r :JuliaCellRun<CR>

" map <Leader>c to execute the current cell
nnoremap <Leader>c :JuliaCellExecuteCell<CR>

" map <Leader>C to execute the current cell and jump to the next cell
nnoremap <Leader>C :JuliaCellExecuteCellJump<CR>

" map <Leader>l to clear Julia screen
nnoremap <Leader>l :JuliaCellClear<CR>

" map <Leader>x to close all Matplotlib figure windows
nnoremap <Leader>x :JuliaCellClose<CR>

" map [c and ]c to jump to the previous and next cell header
nnoremap [c :JuliaCellPrevCell<CR>
nnoremap ]c :JuliaCellNextCell<CR>

" map <Leader>h to send the current line or current selection to Julia
nmap <Leader>h <Plug>SlimeLineSend
xmap <Leader>h <Plug>SlimeRegionSend

" map <Leader>p to run the previous command
nnoremap <Leader>p :JuliaCellPrevCommand<CR>

" map <Leader>q to restart ipython
nnoremap <Leader>q :JuliaCellRestart<CR>

~~~

Note that the mappings as defined here work only in normal mode (see
`:help mapping` in Vim for more information).

Moreover, these mappings will be defined for all file types, not just Python
files. If you want to define these mappings for only Python files, you can put
the mappings in `~/.vim/after/ftplugin/python.vim` for Vim
(or `~/.config/nvim/after/ftplugin/python.vim` for Neovim).

If you come from the MATLAB world, you may want e.g. F5 to save and run the
script regardless if you are in insert or normal mode, F6 to execute the
current cell, and F7 to execute the current cell and jump to the next cell:

~~~vim
" map <F5> to save and run script
nnoremap <F5> :w<CR>:JuliaCellRun<CR>
inoremap <F5> <C-o>:w<CR><C-o>:JuliaCellRun<CR>

" map <F6> to evaluate current cell without saving
nnoremap <F6> :JuliaCellExecuteCell<CR>
inoremap <F6> <C-o>:JuliaCellExecuteCell<CR>

" map <F7> to evaluate current cell and jump to next cell without saving
nnoremap <F7> :JuliaCellExecuteCellJump<CR>
inoremap <F7> <C-o>:JuliaCellExecuteCellJump<CR>

~~~


Supported clipboard programs
----------------------------

Some features of julia-cell use one of the following clipboard programs:

* Linux: [xclip] (preferred) or [xsel].
* macOS: pbcopy (installed by default).
* Windows: not supported.

[xclip]: https://github.com/astrand/xclip
[xsel]: https://github.com/kfish/xsel


FAQ
---

> The `JuliaCellExecuteCell` and `JuliaCellExecuteCellJump` commands do
> not work, but other commands such as JuliaCellRun work. Why?

First, make sure you have Tkinter installed (otherwise you will get an error
message) and a supported [clipboard program](#supported-clipboard-programs).
Also make sure your `DISPLAY` variable is correct, see next question.
If you cannot install the requirements but still want to use the cell execution
feature, you can try the verbose versions `JuliaCellExecuteCellVerbose` and
`JuliaCellExecuteCellVerboseJump`.

> `JuliaCellExecuteCell` and `JuliaCellExecuteCellJump` do not execute the
> correct code cell, or I get an error about
> 'can't open display',
> 'could not open display',
> 'could not connect to display',
> or something similar, what do I do?

Make sure your `DISPLAY` environment variable is correct, especially after
re-attaching a screen or tmux session. In tmux you can update the `DISPLAY`
variable with the following command:

    eval $(tmux showenv -s DISPLAY)

> Should I use `'marks'` or `'tags'` to define cells?

This depends on personal preference. I used to use `'tags'` because they are
similar to MATLAB's `%%` code sections. `'tags'` are nice if you want the cells
to be saved together with your files, and may be easier to start with. I
switched to `'marks'` after discovering that I can show the marks in the
left-most column. I find `'marks'` to be more flexible because you can add and
change cells without changing the code, which is nice when your code is under
version control.

> How do I show the marks in the left-most column?

Use the vim-signature plugin: https://github.com/kshenoy/vim-signature

> How to send only the current line or selected lines to Julia?

Use the features provided by vim-slime, see the
[Example Vim Configuration](#example-vim-configuration) for an example.
The default mapping `C-c C-c` (hold down Ctrl and tap the C key twice) will
send the current paragraph or the selected lines to Julia. See `:help slime`
for more information, in particular the documentation about
`<Plug>SlimeRegionSend` and `<Plug>SlimeLineSend`.

> Why do I get "name 'plt' is not defined" when I try to close figures?

julia-cell assumes that you have imported `matplotlib.pyplot` as `plt` in
Julia. If you prefer to import `matplotlib.pyplot` differently, you can
achieve the same thing using vim-slime, for example by adding the following to
your .vimrc:

    nnoremap <Leader>x :SlimeSend1 matplotlib.pyplot.close('all')<CR>

> How can I send other commands to Julia, e.g. '%who'?

You can easily send arbitary commands to Julia using the `:SlimeSend1`
command provided by vim-slime, e.g. `:SlimeSend1 %who`, and map these commands
to key combinations.

> Why does this plugin not work inside a virtual environment?

If you use Neovim, make sure you have the [neovim] Python package installed.

[neovim]: https://pypi.org/project/neovim/

> Why isn't this plugin specific to Python by default? In other words, why do
> I have to add all this extra stuff to make this plugin Python-specific?

This plugin was created with Python and Julia in mind, but I don't want to
restrict the plugin to Python by design. Instead, I have included examples of
how to use plugin managers to specify that the plugin should be loaded only
for Python files and how to create Python-specific mappings. If someone wants
to use this plugin for other filetypes, they can easily do so.

> Why is this plugin written in Python instead of pure Vimscript?

Because I feel more comfortable with Python and don't have the motivation to
learn Vimscript for this plugin. If someone implements a pure Vimscript
version, I would be happy to consider to merge it.


Related plugins
---------------

* [tslime\_ipython] - Similar to julia-cell but with some small differences.
  For example, tslime\_ipython pastes the whole code that's sent to Julia
  to the input line, while julia-cell uses Julia's `%paste -q` command to
  make the execution less verbose.
* [vim-ipython] - Advanced two-way integration between Vim and Julia. I never
  got it to work as I want, i.e., don't show the code that's executed but show
  the output from the code, which is why I created this simpler plugin.
* [vim-tmux-navigator] - Seamless navigation between Vim splits and tmux panes.
* [vim-signature] - Display marks in the left-hand column.

[tslime\_ipython]: https://github.com/eldridgejm/tslime_ipython
[vim-tmux-navigator]: https://github.com/christoomey/vim-tmux-navigator
[vim-ipython]: https://github.com/ivanov/vim-ipython
[vim-signature]: https://github.com/kshenoy/vim-signature


Thanks
------

julia-cell was heavily inspired by [tslime\_ipython].
The code logic to determine which Python version to use was taken from
[YouCompleteMe].

[tslime\_ipython]: https://github.com/eldridgejm/tslime_ipython
[YouCompleteMe]: https://github.com/Valloric/YouCompleteMe
