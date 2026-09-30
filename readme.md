# sylar

## 开发环境
gcc-9
cmake

## 项目环境
.
|-- bin/            二进制
|-- build/          中间文件路径
|-- cmake/          cmake函数文件夹
|-- CMakeLists.txt  cmake的定义文件
|-- lib/            库的输出路径
|-- Makefile
|-- sylar/          源代码路径
|-- tests/          测试代码路径

## 日志系统
1. 仿照 Log4J 
```
Logger (定义日志类别)
  |
  | ----  Formatter (日志格式)
  |
Appender (日志输出地方)

```

### 调试
1. 在编译选项中加入`-O0 -ggdb`, 启用调试
2. 进入gdb: `gdb ./bin/test`
```sh
$ gdb bin/test 
GNU gdb (Ubuntu 15.1-1ubuntu1~24.04.1) 15.1
Copyright (C) 2024 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<https://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Reading symbols from bin/test...

(gdb) b sylar::Logger::log(sylar::LogLevel::Level, std::shared_ptr<sylar::LogEvent>) # 增加断点
Breakpoint 1 at 0x6140

(gdb) r # run, 执行程序,直到遇到第一个断点
Starting program: /home/lin/data/github/slyar-learning/bin/test 

This GDB supports auto-downloading debuginfo from the following URLs:
  <https://debuginfod.ubuntu.com>
Enable debuginfod for this session? (y or [n]) n
Debuginfod has been disabled.
To make this setting permanent, add 'set debuginfod enabled off' to .gdbinit.
[Thread debugging using libthread_db enabled]
Using host libthread_db library "/lib/x86_64-linux-gnu/libthread_db.so.1".

Breakpoint 1, sylar::Logger::log (this=0x5555555722b0, level=sylar::LogLevel::DEBUG, event=std::shared_ptr<sylar::LogEvent> (use count 2, weak count 0) = {...})
    at /home/lin/data/github/slyar-learning/sylar/log.cc:217
217     void Logger::log(LogLevel::Level level, LogEvent::ptr event) {

(gdb) p level # 打印变量值
$1 = sylar::LogLevel::DEBUG

(gdb) p m_level # 打印变量值
$2 = sylar::LogLevel::DEBUG

(gdb) n # next, 下一步(不进入函数内部)
218         if (level >= m_level) {

(gdb) n # next, 下一步(不进入函数内部)
219             auto self = shared_from_this();

(gdb) n # next, 下一步(不进入函数内部)
220             for (auto &i : m_appenders) {

(gdb) p m_appenders # 打印变量值
$3 = std::__cxx11::list = {[0] = std::shared_ptr<sylar::LogAppender> (use count 1, weak count 0) = {get() = 0x5555555723c0}}

(gdb) n # next, 下一步(不进入函数内部)
221                 i->log(self, level, event);

(gdb) s # step, 单步执行(会进入函数内部)
std::__shared_ptr_access<sylar::LogAppender, (__gnu_cxx::_Lock_policy)2, false, false>::operator-> (this=0x555555572420) at /usr/include/c++/13/bits/shared_ptr_base.h:1357
1357            return _M_get();

(gdb) fin # finish, 执行完当前函数，然后停住
Run till exit from #0  std::__shared_ptr_access<sylar::LogAppender, (__gnu_cxx::_Lock_policy)2, false, false>::operator-> (this=0x555555572420) at /usr/include/c++/13/bits/shared_ptr_base.h:1357
0x00007ffff7fa0d4c in sylar::Logger::log (this=0x5555555722b0, level=sylar::LogLevel::DEBUG, event=std::shared_ptr<sylar::LogEvent> (use count 2, weak count 0) = {...})
    at /home/lin/data/github/slyar-learning/sylar/log.cc:221
221                 i->log(self, level, event);
Value returned is $4 = (std::__shared_ptr_access<sylar::LogAppender, (__gnu_cxx::_Lock_policy)2, false, false>::element_type *) 0x5555555723c0

(gdb) b sylar::StdoutLogAppender::log(std::shared_ptr<sylar::Logger>, sylar::LogLevel::Level, std::shared_ptr<sylar::LogEvent>) # 新增断点
Breakpoint 2 at 0x7ffff7fa13fe: file /home/lin/data/github/slyar-learning/sylar/log.cc, line 270.

(gdb) c # 继续执行,直到遇到下一个断点或程序结束/崩溃
Continuing.

Breakpoint 2, sylar::StdoutLogAppender::log (this=0x5555555723c0, logger=std::shared_ptr<sylar::Logger> (use count 3, weak count 1) = {...}, level=sylar::LogLevel::DEBUG, 
    event=std::shared_ptr<sylar::LogEvent> (use count 3, weak count 0) = {...}) at /home/lin/data/github/slyar-learning/sylar/log.cc:270
270     {

(gdb) n # next, 下一步(不进入函数内部)
271         if (level >= m_level) {

(gdb) p level # 打印变量值
$5 = sylar::LogLevel::DEBUG

(gdb) p m_formatter # 打印变量值
$6 = std::shared_ptr<sylar::LogFormatter> (use count 2, weak count 0) = {get() = 0x555555572320}

(gdb) b sylar::LogFormatter::format(std::shared_ptr<sylar::Logger>, sylar::LogLevel::Level, std::shared_ptr<sylar::LogEvent>) # 新增断点
Breakpoint 3 at 0x7ffff7fa1590: file /home/lin/data/github/slyar-learning/sylar/log.cc, line 285.

(gdb) c # 继续执行,直到遇到下一个断点或程序结束/崩溃
Continuing.

Breakpoint 3, sylar::LogFormatter::format[abi:cxx11](std::shared_ptr<sylar::Logger>, sylar::LogLevel::Level, std::shared_ptr<sylar::LogEvent>) (this=0x555555572320, 
    logger=std::shared_ptr<sylar::Logger> (use count 4, weak count 1) = {...}, level=sylar::LogLevel::DEBUG, event=std::shared_ptr<sylar::LogEvent> (use count 4, weak count 0) = {...})
    at /home/lin/data/github/slyar-learning/sylar/log.cc:285
285     {

(gdb) n # next, 下一步(不进入函数内部)
286         std::stringstream ss;

(gdb) p m_items # 打印变量值
$7 = std::vector of length 0, capacity 0

(gdb) n # next, 下一步(不进入函数内部)
287         for (auto& i : m_items) {
(gdb) n # next, 下一步(不进入函数内部)
290         return ss.str();
```

## 协程库封装

## socket函数库

## http协议开发

## 分布式协议库

## 推荐系统