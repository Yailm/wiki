### CtrlP

* `<c-p>` open the CtrlP prompt in find file mode.  
  * `<c-d>` toggle between full-path search and filename only search
  * `<c-r>` toggle between the string mode and full regexp mode
  * `<c-f>` `<c-up>` scroll to the 'next' search mode in sequence
  * `<c-b>` `<c-down>` scroll to the 'previous' search mode in the sequence
  * `<tab>` auto-complete directory names under the current working directory inside the prompt
  * `<s-tab>` toggle the focus bwtween the match window and the prompt
  * `<esc>` `<c-c>` exit
  * 
  * --- move ---
  * `<c-j>` `<down>` move selection down
  * `<c-k>` `<up>` move selection up
  * `<c-a>` move the cursor to the 'start' of the prompt
  * `<c-e>` move the cursor to the 'end' of the prompt
  * `<c-h>` `<left>` `<c-^>` move the cursor one character to the 'left'
  * `<c-l>` `<right>` move the cursor one character to the 'right'
  * 
  * --- edit ---
  * `<c-]>` `<bs>` delete the preceding character
  * `<del>` delete the current character
  * `<c-w>` delete a preceding inner word
  * `<c-u>` clear the input field
  * 
  * --- browsing input history ---
  * `<c-n>` next string in the prompt's history
  * `<c-p>` previous string in the prompt's history
  * 
  * --- open/create a file ---
  * `<cr>` open the selected file in the 'current' window if possible
  * `<c-t>` open the selected file in a new 'tab'
  * `<c-v>` open the selected file in a 'vertical' split
  * `<c-x>` `<c-cr>` `<c-s>` open the selected file in a 'horizontal' split
  * `<c-y>` create a new file and its parent directories
  * 
  * --- open multiple files ---
  * `<c-z>`
    * mark/unmark a file to be opened with `<c-o>`
    * mark/unmark a file to create a new file in its directory using `<c-y>`
  * `<c-o>`
    * open files marked by `<c-z>`
    * when no file has been marked by `<c-z>`, open a console dialog with the follwoing options
      * open the selected file
        - t - in a tab page
        - v - in a vertical split
        - h - in a horizontal split
        - r - in the current window
        - i - as a hidden buffer
        - x - (optional) with the function defined in g:ctrlp_open_func
      * other options (not shown)
        - a - mark all files in the match window
        - d - change CtrlP's local working directory to the selected file's directory and switch to find file mode
  *
  * `<F5>`
    - refresh the match window and purge the cache for the current directory
    - remove deleted files from the MRU list.
  * `<F7>`
    * MRU mode:
      - wipe the list
      - delete entries marked by `<c-z>`
    * buffer mode:
      - delete entry under the cursor or delete multiple entries marked by `<c-z>`
  * --- paste ---
  * `<Insert>` `<MiddleMouse>` paste the clipboard content into the prompt
  * `<c-\>` open a console dialog to paste `<cword>`, `<cfile>`, the content of the search register, the last visual selection, the clipboard or any register into the prompt.
