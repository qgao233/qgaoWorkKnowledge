## 2 idea等ide的build区别

* compile：只编译选定的目标。
* make：只编译选定的目标，只编译上次编译后，产生过变化的文件。
* build：对整个工程进行编译。

但现在都用第三方工具进行源码编译的管理，如：ant,maven等工具。
>即maven的package、install等已经替代了ide提供的build。