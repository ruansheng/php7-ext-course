## 扩展内存泄露检查

### 
```

测试:
php -r 'var_dump(mypow());'
bool(true)
[Sat Dec 30 00:58:34 2017]  Script:  'Standard input code'
/usr/local/php7/include/php/Zend/zend_string.h(134) :  Freeing 0x00007fcbfd401800 (32 bytes), script=Standard input code
=== Total 1 memory leaks detected ===
```
