### 单例模式

##### 作用

- 用于为一个类生成一个唯一的对象。
- 最常用的地方是数据库连接、日志输出等。

##### 特点

- 一个类只能有一个实例。
- 不可被覆盖和克隆。
- 不需要直接实例。

   ##### 事例

```php
class Sigle{

    // 方法不可被覆盖
    final protected function __construct()
    {
    }

    private static $ins = null;

	public static function getIns(){

		if(is_null(self::$ins)){
            self::$ins = new self();
		}

		return self::$ins ;
	}

	// 防止克隆
	final protected function __clone()
    {
        // TODO: Implement __clone() method.
    }

  
  public function test(){
    echo "单例调用";
  }
}

$Sigle = Sigle::getIns();
$Sigle->test();
```



### 工厂模式

##### 作用：

​	众多子类并且会扩充、创建方法比较复杂。

##### 区别

- 简单工厂模式(静态方法工厂模式) ： 用来生产同一等级结构中的任意产品。（不能增加新的产品）
- 工厂模式 ：用来生产同一等级结构中的固定产品。（支持增加任意产品）
- 抽象工厂 ：用来生产不同产品种类的全部产品。（不能增加新的产品，支持增加产品种类）

##### 实现

- 简单工厂模式：是通过一个`静态方法`来创建对象的。

```php

/**
 * 动物的接口
 * Interface zoom
 */
interface zoom{
    public function call();
}

/**
 * 狗 继承了动物的类
 * Class dog
 */
class dog implements zoom{
    public function call(){
        echo "汪汪汪";
    }
}

/**
 * 猫 继承了动物的类
 * Class cat
 */
class cat implements zoom{
    public function call(){
        echo "喵喵喵";
    }
}

/**
 * 动物的工厂
 * Class ZoomFactory
 */
class ZoomFactory
{
    // 生产一个狗
    static function createdDog(){
        return new dog();
    }

    // 生产一直猫
    static function createdCat(){
        return new cat();
    }
}

$dog = ZoomFactory::createdDog();
$dog ->call();
$cat = ZoomFactory::createdCat();
$cat ->call();
```

- 工厂模式：定义一个用于创建对象的接口，让子类决定哪个类实例化。 他可以解决`简单工厂模式`中的封闭开放原则问题。

```php

/**
 * 动物的接口
 * Interface zoom
 */
interface zoom{
    public function call();
}

/**
 * 狗 继承了动物的类
 * Class dog
 */
class dog implements zoom{
    public function call(){
        echo "汪汪汪";
    }
}

/**
 * 猫 继承了动物的类
 * Class cat
 */
class cat implements zoom{
    public function call(){
        echo "喵喵喵";
    }
}

/**
 * 动物创建的接口
 * Interface createdZoom
 */
interface createdZoom{
    public function created();
}

/**
 * 生成一个狗的工厂 继承动物创建的接口
 * Class DogFactory
 */
class DogFactory implements createdZoom{
    public function created(){
        return new dog();
    }
}

/**
 * 生成一个猫的工厂 继承动物的创建的接口
 * Class CatFactory
 */
class CatFactory implements createdZoom{
    public function created(){
        return new cat();
    }
}

$dog = (new DogFactory())->created();
$dog ->call();
$cat = (new CatFactory())->created();
$cat ->call();
```

- 抽象工厂模式

```php
/**
 * 动物接口
 * Interface zoom
 */
interface  zoom
{
    public function  color();
}

/**
 * 白色狗狗 继承zoom
 */
class WhiteDog implements zoom
{
    public function color()
    {
        echo '白色狗狗';
    }
}

/**
 * 黑色狗狗 继承zoom
 */
class BlackDog implements zoom{
    public function color()
    {
        echo '黑色狗狗';
    }
}

/**
 * 白色猫咪 继承zoom
 */
class WhiteCat implements zoom {
    public function color()
    {
        echo '白色猫咪';
    }
}

/**
 * 黑色猫咪 继承zoom
 */
class BlackCat implements zoom {
    public function color()
    {
        echo '黑色猫咪';
    }
}

/**
 *  创建动物对象类
 * 注意:这里将对象的创建抽象成了一个接口。
 */
interface  createZoom
{
    public function createWhite();

    public function createBlack();

}

/**
 *  创建狗狗对象的工厂类
 */
class DogFactory implements createZoom
{
    // 创建白色狗狗
    public function createWhite()
    {
        return new WhiteDog();
    }

    // 创建黑色狗狗
    public function createBlack()
    {
        return new BlackDog();
    }
}

/**
 * 创建猫咪对象的工厂类
 */
class FactoryWomen implements createZoom
{
    // 创建第一个女人
    public function createWhite()
    {
        return new WhiteCat();
    }

    // 创建第二个女人
    public function createBlack()
    {
        return new BlackCat();
    }
}

// 狗狗
$factory = new DogFactory();
$man = $factory->createWhite();
$man->color();

$man = $factory->createBlack();
$man->color();

// 猫咪
$factory = new FactoryWomen();
$man = $factory->createWhite();
$man->color();

$man = $factory->createBlack();
$man->color();
```



- 