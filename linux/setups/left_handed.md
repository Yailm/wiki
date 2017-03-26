### Linux Left Handed Mouse

- 首先切换鼠标的左右按键

      echo 'pointer = 3 2 1' >> ~/.Xmodmap

  确认在**.xinitrc**中有将.Xmodmap加载

- 更改鼠标的样式（让指针向右）

  查看系统的主题

      cat ~/.icons/default/index/theme
      cat /usr/share/icons/defauit/index.theme

  找到在/usr/share/icons/(或~/.icons/)下的主题目录，此时我是Adwaita

      cd /usr/share/icons/Adwaita/cursors/

  里面有left_ptr和right_ptr，交换里面的内容就行了

      mv left_ptr /tmp
      mv right_ptr left_ptr
      mv /tmp/left_ptr .

- see also

    [https://wiki.archlinux.org/index.php/Cursor_themes](https://wiki.archlinux.org/index.php/Cursor_themes)
