
copy/paste in urxvt
------------------------------------

add following lines into `$HOME/.Xresource`

    URxvt.perl-ext-common: default,keyboard-select,clipboard
    URxvt.keysym.M-Escape: perl:keyboard-select:activate
    URxvt.keysym.M-s: perl:keyboard-select:search
    URxvt.keysym.M-c: perl:clipboard:copy

then install `urxvt-perls`

    $ pacaur -S urxvt-perls

enjoy urxvt with `alt-c` `alt-v` and `alt-escape`

but it will be more complicated with using `urxvtc`
