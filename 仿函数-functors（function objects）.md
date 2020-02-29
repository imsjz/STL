## 仿函数-functors（function objects）

* 仿函数可配接的关键（adaptable）

  `STL`定义了个两个`classes`，分别代表一元仿函数和二元仿函数。如果想要自己写的仿函数有可配接的功能，就要选择继承其中一个。

  * 一元仿函数源码如下：

    ```cpp
    // 一元仿函数
    template <class _Arg, class _Result>
    struct unary_function {
      typedef _Arg argument_type;
      typedef _Result result_type;
    };
    ```

  * 二元仿函数实现如下：

    ```cpp
    // 二元仿函数
    template <class _Arg1, class _Arg2, class _Result>
    struct binary_function {
      typedef _Arg1 first_argument_type;
      typedef _Arg2 second_argument_type;
      typedef _Result result_type;
    };      
    ```

    