## 扩展中加载执行php文件
在PHP中提供了包含文件的语法: include、include_once、require、require_one ,下面看一下每个语句的作用与特点
include : 包含并运行指定文件，未找到文件则 include 结构会发出一条警告，脚本会继续运行
include_once: 功能和include相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。
require : 包含并运行指定文件，基本同include，只是在出错时产生 E_COMPILE_ERROR 级别的错误，将导致脚本中止
require_once : 功能和require相同，唯一区别是 PHP 会检查该文件是否已经被包含过，如果是则不会再次包含。

### 内核执行PHP文件的流程

