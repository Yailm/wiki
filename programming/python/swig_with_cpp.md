### 使用swig将c++项目转化成python可用的模块

1. 编写swig interface文件，包含需要导出的类和函数

    /* example.i */
    %module example
    %{
    #include "example.h"
    %}
    %include "example.h"
    /*
     * 假设你需要的是example.cpp文件中的类和方法
     * 而它正好有example.h文件，就能把接口文件直接这样写
     */

2. 执行`swig -c++ -python example.i`，这会生成两个文件**example.py**和**example_wrap.cxx**

3. 将工程下**所有相关**的cpp或cxx文件都编译了

    g++ -O2 -fPIC -c *.cpp
    # 对于特殊的example_wrap.cxx文件
    g++ -O2 -fPIC -c example_wrap.cxx -I/usr/include/python3.5m

4. 添加上一部生成的所有.o文件，生成动态连接库，还需要加上依赖

    g++ -shared *.o exmaple.o example_wrap.o -o _example.so

5. example.py就是我们最后的模块，它会自动调用_example.so动态库（名字是有关系的）

    import example  # 没有错误就大功告成

#### 从python中传递list到c的array

- 最后不要使用`%include "carrays.i"生成的对象，速度太慢
- 不推荐使用复杂的typemap函数
- 使用numpy项目提供的[numpy.i](https://raw.githubusercontent.com/numpy/numpy/master/tools/swig/numpy.i)，对普通的list也能使用

Example:

    /* kcf.i */
    %module kcf
    %{
    #define SWIG_FILE_WITH_INIT
    #include "kcf.h"
    %}
    %include "numpy.i"
    %init %{
    import_array();
    %}
    %apply(unsigned char * IN_ARRAY1, int DIM1) {(unsigned char * _img, int n)}
    %include "kcf.h"

    /* kcf.h */
    ...
        // Init/re-init methods
        void init(unsigned char * _img, int n, int _x, int _y, int w, int h);
        void updateTrackerPosition(BBox_c & bbox);

        // frame-to-frame object tracking
        void track(unsigned char * _img, int n);
    ...

在python中，就都能直接传入数组了

    from kcf import *
    kcftracker = KCF_Tracker()
    kcftracker.init([12,34,123,23,..], 12, 22, 33, 44)
    kcftracker.tracker(image.flatten().tolist())
