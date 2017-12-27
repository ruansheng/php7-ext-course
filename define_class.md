## 定义类
在扩展中定义的类，在我们经常写的.php文件中，可以调用

### 类的修饰
PHP中class可以指定一些修饰符，这些修饰符分别是: final、abstract
final: 当final修饰class的时候，此类被限定为不可继承类，也就是其他类无法继承此类，被称为最终类
       当你不想让别人继承自己的编写的类时只需要在前面加上final关键字即可
abstract: 当abstract修饰class的时候，此类被限定为抽象类，只能用于继承，而无法实例化对象，也就是不能new class。     
