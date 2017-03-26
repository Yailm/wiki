### service

每个service是thing的直接子目录，里面必须包含`service.json`文件，还可以其他的js文件

#### 各个文件描述

| 文件名             | 描述                  |
|--------------------|-----------------------|
| service.json       | 描述service的一些属性 |
| service_init.js    | 初始化service         |
| service_destroy.js | 摧毁service           |
| start.js           | 启动并初始化会话      |
| resume.js          | 恢复会话              |
| after_resume.js    | 在会话恢复后的操作    |
| pause.js           | 暂停会话              |
| stop.js            | 摧毁关闭会话          |

如果上面的文件缺失，系统会自动使用内存中默认的文件（除了service.json）

#### 状态

- service状态有两个，**已经初始化**或**没有**
- session(会话)状态：闲置、暂停和工作

| current state | start   | resume  | pause   | stop    |
|---------------|---------|---------|---------|---------|
| idle          | paused  | invalid | invalid | invalid |
| paused        | invalid | working | invalid | idle    |
| working       | invalid | invalid | paused  | invalid |

#### service.json字段

* id: 字符串，service的唯一标识。默认值: one created by service directory path and thing id.
* name: 字符串，service名字
* description: 字符串，service描述
* spec: 必需，字符串或对象，如果该字段是字符串，它表示service链接到的spec对象
* config: 对象，service的配置信息

#### JS files

上面列表中提到的js文件都运行在沙箱中，并提供一些全局变量

- `shared`
  这是在单个会话中的全局对象，`start.js`, `resume.js`, `after_resume.js`, `kernel.js`, `pause.js`, `stop.js`都会共享这个对象。举个例子，start.js中的`shared.num = 1`，在stop.js我们能够使用这个变量
- `service_shared`
  service中共享的全局对象，`start.js`, `resume.js`, `after_resume.js`, `kernel.js`, `pause.js`, `stop.js`, `service_init.js`, `service_destroy`。所以同一服务中运行的两个会话都能够共享这个对象
- `hub_shared`
  在hub中共享的全局对象
- `CONFIG`
  全局的只读对象，包含了service.json中的数据
- `IN`
  全局的只读对象，用在`kernel.js`，存在会话进入口中的数据类似{in1: value, in2: value}
- `done(value)` and `fail(err)`
  它们都是全局函数用在`start.js`, `resume.js`, `pause.js`, `stop.js`, `service_init.js`, `service_destroy.js`，会让服务和会话的转换
  - `done(value)`当操作成功完成时调用，value值会被记录下来
  - 多次使用这两个全局方法时，只有最开始的被执行，后面的内容都被忽略
  - 当任何异常发生的时候，它会自动调用`fail(value)`
- `sendOUT(out)` and `sendERR(err)`
  它们都是全局函数用在`after_resume.js`和`kernel.js`中，只用在会话的状态是工作的时候，这些函数才有效
  - `sendOUT(out)`会发送out到会话的出口，而out是键值对，`e.g {o1: value1, o2: value2}`，其中o1、o2是出口的名字
  - `sendERR(err)`会发送err到会话的错误出口处
  - 当任何错误发生的时候，它会自动调用`sendERR(exception)`

##### 默认文件内容

如果任何service的js文件缺失，系统会自动在内存中创建默认的内容

- `after_resume.js` and `kernel.js`，默认为空
- `start.js`, `resume.js`, `pause.js`, `stop.js`, `service.init.js`, `service_destroy.js`，默认内容为`done()`

##### 执行顺序

- 当工作流启动的时候，所有的会话都会先执行`start.js`，当操作完成后，随后执行`resume.js`，之后就会执行`after_resume.js`
- 当工作流暂停的时候，所有的会话执行`pause.js`
- 当工作流恢复执行，所有的会话`resume.js`，之后再执行`after_resume.js`
- 当工作流停止时，如果在工作的话，所有的会话执行`pause.js`，暂停操作完成后继续执行`stop.js`；而如果本来工作流已经暂停，则直接执行`stop.js`
- 当会话被触发的时候，这个时候执行`kernel.js`
- 当会话准备执行`start.js`时，如果该服务还没有初始化，那么将先执行`service_init.js`
