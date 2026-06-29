# fznote
Plain-Text Knowledge Base Engine
markdown
# Plain-Text Knowledge Base Engine

A fast, zero-dependency knowledge retrieval and note-editing engine
built purely on standard UNIX primitives (`fzf` and `less`). 

## The Philosophy

Modern software engineering is plagued by app bloat. Developers rely on heavy
graphical note-taking apps, relational databases, or complex web services just
to store commands, code snippets, and technical help files. 

This repository provides an alternative workflow honed over years of daily
terminal use. It treats a single, ever-expanding plain-text file as an infinite
database. By pairing `fzf` for macro-searching with `less` for micro-inspection
and native text editing, you create a blazing-fast, write-accessible knowledge
ecosystem that runs entirely inside your shell runtime.

It is simple, incredibly fast, consumes virtually zero system resources, and gives you a panoramic of your knowledge base. It allows you to instantly find your decade-old notes and command snippets so you can review, update or copy out what you need for re-use in new scripts.

## The Core Concept: The "Master-Detail" Loop

The engine relies on an unstructured text format where technical snippets are
separated by arbitrary tag headers beginning with an underscore (`_`). 

When executed, the system creates a seamless, infinite loop:
1. **Macro Search (`fzf`)**: Interactively filter through your custom tag headers.
2. **Micro Inspection (`less`)**: Selecting a tag drops you directly onto that specific line in your massive file using `less +/"_tag"`.
3. **On-the-Fly Modification (`v`)**: Pressing `v` inside `less` forks instantly into your system editor (e.g., Vim) at that exact line for seamless updates.
4. **Context Maintenance**: Quitting the editor returns you to `less`. Quitting `less` returns you right back to your active `fzf` search viewport.

---

## Installation & Setup

1. Ensure you have `fzf`i and xclip installed on your system.
2. Add the following function to your shell configuration file (e.g., `~/.bashrc` or `~/.zshrc`).

```
function ulc () {
    # Set default paths and variables
    local UT_PATH=~/utags
    local FILENAME="${1:-}"  # First argument is the input file, default to empty
    
    # Help function
    show_help() {
        cat << EOF
Usage: ulc [OPTIONS] [FILENAME]
ulc                             Uses default file at ~/utags
ulc /path/to/your/tags.txt      Uses argument as input file
ulc -h or ulc --help            Shows help information

Description:
    Interactive review of personal tagged knowledge book

Options:
    -h, --help    Show this help message
    FILENAME      Path to the input file (optional, defaults to ~/learn/linux/utags)

Key Bindings:
    ENTER         Launch knowledge book in less - scrolled to show the selected tag
    ESC           Exit the fzf interface
    q (in less)   Return from less view
    v (in less)   launch in editor
Example:
    ulc                           # Uses default UTAGS file
    ulc /path/to/my/notes.txt      # Uses specified file
    ulc -h                        # Shows this help
# WARNING: DO NOT indent the line below! "EOF" must start at the very beginning of the line.
EOF
    }
    
    # Check for help flag
    if [[ "${1}" == "-h" ]] || [[ "${1}" == "--help" ]]; then
        show_help
        return 0
    fi
    
    # If no filename provided, use default
    if [[ -z "${FILENAME}" ]]; then
        FILENAME="${UT_PATH}"
    fi
    
    # Check if file exists
    if [[ ! -f "${FILENAME}" ]]; then
        echo "Error: File '${FILENAME}' not found"
        return 1
    fi
    
    # Main fzf command with preview and execute
    sed -n '/^_/p' "${FILENAME}" | \
        fzf \
            --preview-window='up:wrap' \
            --preview "sed -n '/^{}/,/^_/p' '${FILENAME}' | head -n -1" \
            --bind="enter:execute(less -p '^{}' '${FILENAME}')" \
            --header=$'Press: ENTER to view in less; ESC to exit \nq (while viewing) to return'
}
```
The script assumes your notes are in a file named utags is in home directory. You can ignore this default behavior and simply pass the full pathname of your notes file  as argument to the ulc function.  For example:
ulc ~/folder1/folder2/my-notes.txt

But if you name your file utag and keep it in your home directory then you can call the function with no arguments:
ulc

---

## How to Format Your Notes File

There are no rigid structural guidelines or chronologically ordered folders
required. You append new discoveries anywhere in your text file. 

The engine only requires that every block of notes begins with a line starting
with an underscore followed a combination of whatever words that would jog your memory later, all concatinated as a single word. For example if your note is about for-loops in awk function it would be sensible to choose something like _awk-for-loop or _awk-ForLoop. Use as many words as you wish to help fzf to make a hit.

### Example File Structure (`notes.txt`)

```text
_vim-LineNavigation
  12G     Move cursor to line 12
  32j     Go 32 lines down
  21k     Go 21 lines up
  H       Move cursor to line at the top of the window
  M       Move cursor to the line at the middle of the window
  L       Move cursor to the line at the bottom of the window
   
  paragraph: Blocks of consecutive non-empty lines.
  :h paragraph to learn more
  }       Move cursor to next paragraph
  {       Move cursor to previous paragraph

_tar-compression-examples
  tar -cvf archive.tar /path/to/directory      Create uncompressed tar
  tar -xvf archive.tar                         Extract tar archive
  tar -czvf archive.tar.gz /path/to/dir        Create gzip compressed tar
  tar -xzvf archive.tar.gz                     Extract gzip compressed tar

_iptables-block-ip-address
  iptables -A INPUT -s 1.2.3.4 -j DROP         Block a specific IP address
  iptables -D INPUT -s 1.2.3.4 -j DROP         Unblock a specific IP address
```
To test the engine you can download the utag file in this repo which is a random section of my main tags file.

## Why This Method Wins

- **Performance**: Scales flawlessly. A 7,000+ line note file queries and loads in milliseconds because it relies on raw memory and streaming text buffers.
- **Portability**: Syncs seamlessly across multiple machines via (private) Git repositories without any data format incompatibilities.
- **Future-Proof**: Relies strictly on plain text. It will remain fully functional and readable decades from now, independent of any corporatesoftware ecosystems.
---------------------------------------------------------


markdown
## Advanced Feature: Command Injection

For a frictionless terminal workflow, you can bind a shortcut to a function (see below)
that puts clipboard contents (xclip) onto the command line. 
The function uses Bash internal runtime mechanics (`READLINE_LINE`) for this operation.


### Putting clipboard contents on command line

Add this secondary function and its keybinding to your `~/.bashrc` file:

```
# Replace entire command line with xclip contents                               
paste_xclip() {                                                                 
    READLINE_LINE="$(xclip -o)"                                                 
    READLINE_POINT=${#READLINE_LINE}                                            
}                                                                               
#use ATL+q to inject xclip contents to command line: 
bind -x '"\eq": "paste_xclip"'

```
Pressing ALT+q will dump xclip contents on the command line.
The copied line will often include your annotations and your intended command. So editing is unavoidable.

---
## Installation
No formal installation is required. Simply add a copy of the main engine (the function called ulc) to your .bashrc. Optionally also add (to .bashrc) the past_xclip function and its binding as shown above. Source your .bashrc after editing. And lastly have your notes file -- tagged personal knowledge text file -- handy. If you don't have one yet you can make a start by copying the sample list provided above. Save that somewhere under any name (e.g. utags) and launch the engine with:
ulc ~/folder1/folder2/../utags


---
## System requirements:
fzf
xclip
Any Linux distro

---
## Tips on tag construction
Construct a long tag word that contains as many words as possible that is relevant to your note entry. This makes the job of fzf much easier as you later type those words -- in random order regardless of spacing etc. fzf will display all tags that contain those words and you will instantly see which one you meant. It take fzf almost no time (even on a slow laptop) to find and display those tags from 10,000 lines of text. As you move from selection to selection the contents of that tags appear in the preview window. If it is the right one then press ENTER and the whole tag file is launched in 'less' -- the file scrolls to display the selected tag. If it is not the right one simply press q and leave 'less' and you are back in the tag selection mode.
For example you might have written some notes about how to mark lines and copy them in 'less'. Choose a tag that is something like \_less-mark-copy. When you launch the engine you do not need to remember the exact tag to find that note. Just type as many of the words that you can remember in any order:
less copy
less mark
mark copy less
.
.

This is the main insight of this tool. You can have as many tags as you like without having to remember any of them. Just the ones that might have sensibly expressed your note entry. I have notes going back more than ten years and I can find them in seconds.

## Copy operations in 'less'
This engine uses xclip as the preferred clipbaord.
the process of copying lines of text in 'less' is fast and simple. Here is a brief description of the basics.
Suppose you want to copy a block of lines to xclip. Scroll until the bottom line of the block is the last visible line. Press M (capital!) and choose a marker, e.g. the letter a. Now scroll again until the top line of your intended block is the first visible line on the screen. Now press | and you will be asked for your marker. Press a and you will see ! Now  type xclip and press ENETR and you are done. Those line are now in xclip. You can check that from terminal by running xclip -o.

This is just one way of copying lines in 'less'. There are a shortcut when you want just one line copied. No need for a marker in this case. Scroll and bring that line to the top of the visible screen and then:
press |
Ignore the request for marker and press ENETR 
! appears. Now type:
head -1 | xclip (press ENTER) # and you are done 

head -2 | xclip would have copied two line from the top and so on.

Marking and processing lines of text in 'less' is a very interesting and important topic and is well worth your while to learn. Copying is just one option of processing the text lines which we have leveraged for this engine.

## Editor
'less' gives you access to your favourite editor by pressing the letter v
Test it by opening a file in 'less' and pressing v. Nano may show up as you editor. If you are happy with that then no action needed. You can change it (e.g to vim)  by adding these two lines to your .bashrc file:

export VISUAL="vim"
export EDITOR="vim" 
and then:
source ~/.bashrc

Make changes if you need to, save and exit the editor. You are then back in 'less' with the updated knowledge file. You can leave 'less' (press q) and you are back in fzf loop -- also wih the updated file in display


---

## License

This project is open-source and free to use. Build your own knowledge, play
with your own rules, and keep your terminal lean and clean.
