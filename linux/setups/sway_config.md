### Sway 配置全解读

* 只能在配置文件中使用的项

  * bar &lt;block or command>，具体详细见sway-bar(5)

  * set &lt;name> &lt;value>，设置变量

  * orientation &lt;horizontal|vertical|auto>，设置默认的分割布局形式

* 只能跟在bindsym或者在运行时通过swaymsg设置

  * border &lt;normal|pixel> [&lt;n>]，设置边框的样式，<u>normal</u>是包括n像素宽边框和标题栏，<u>pixel</u>是n像素宽边框不含标题栏

  * border &lt;none|toggle>，设置边框的样式或在<u>normal</u>, <u>pixel</u>, <u>none</u>中切换

  * exit

  * floating &lt;enabled|disable|toggle>，使当前focused窗口浮动，取消浮动，或切换成另一种状态

  * focus &lt;direction>，方向可以是**up**, **down**, **right**, **left**, **parent**或**child**。
    parent用在移动整个容器非常有效，[see also](https://i3wm.org/docs/userguide.html#_focus_parent)

  * focus output &lt;direction>

  * focus mode_toggle 在平铺和浮动视图切换**焦点**，焦点移动的时候，是不能跨过两种不同的视图的，就需要这快捷键

  * fullscreen

  * layout &lt;splith|splitv|toggle split|stacking|tabbed>，为当前的容器设置布局

  * move &lt;left|right|up|down>

  * move &lt;container|window> to workspace &lt;name>，将当前的容器或窗口移动到标记为name的工作区

  * move &lt;container|window|workspace> to output &lt;name|direction>

  * reload，重新加载配置文件

  * resize &lt;shrink|grow> &ltwidth|height> [&lt;amount>] [px|ppt]，按amount调整当前容器或视图，px指单位像素，ppt指单位为当前窗口尺寸的百分比

  * resize set &lt;width [px] &lt;height> [px]

  * resize set &lt;width|height> &lt;amount> [px] [&lt;width|height> &lt;amount> [px]]

  * split &lt;vertical|v|horizontal|h|toggle|t>
  * splith
  * splitv
  * splitt

  * sticky &lt;enable|disable|toggle>，让一个**浮动**窗口在所有工作区中显示

* 下面命令既可用于配置文件，也可以在运行时设置

  * assign &lt;crtiteria> [-] &lt;workspace>

  * bindsym &lt;key combo> &lt;command>

  * client &lt;color_class> &lt;border> &lt;background> &lt;text> &lt;indicator> &lt;child_border>

  * debuglog &lt;on|off|toggle>，不可用在配置文件中

  * exec &lt;shell command>，用来执行shell命令

  * exec_always &lt;shell command>，像exec，不过该命令还会在<u>reload</u>和<u>restart</u>执行一次

  * floating_maximum_size &lt;width> x &lt;height>，指定浮动窗口的最大尺寸。默认使用容器的尺寸，`-1 x -1`会移除任何限制，如果与<u>floating_minimum_size</u>冲突，最大尺寸优先

  * floating_minimum_size &lt;widtht> x &lt;height>，默认是75 x 50，0和-1都是无效的

  * floating_modifier &lt;modifier> [normal|inverse]，与浮动窗口的鼠标事件有关

  * floating_scroll &lt;up|down|left|right> [command]，modifier键按下时滚动执行的命令

  * focus_follows_mouse &lt;yes|no>

  * for_window &lt;criteria> &lt;command>

  * gaps edge_gaps &lt;on|off|toggle>，暂时没弄懂

  * gaps &lt;inner|outer> &lt;amount>

  * gaps &lt;inner|outer> &lt;all|workspace|current> &lt;set|plus|minus> &lt;amount>
