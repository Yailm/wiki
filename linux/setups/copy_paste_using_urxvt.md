
copy/paste in urxvt
------------------------------------

add following lines into `$HOME/.Xresource`

    URxvt.perl-ext-common: default,clipboard
    urxvt.keysym.M-c:   perl:clipboard:copy
    urxvt.keysym.C-v:   perl:clipboard:paste
    urxvt.keysym.M-C-v: perl:clipboard:paste_escaped

then install `urxvt-perls`

    $ pacaur -S urxvt-perls

enjoy urxvt with `alt-c` and `ctrl-v`
