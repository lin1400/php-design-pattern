# PHP 设计模式笔记

###设计模式简介

基础的三种设计模式：

1.  工厂模式

2. 单例模式

3. 注册模式

> 设计模式主要是为了设计程序代码，让代码实现的更加智能化，解耦合，修改起来更加方便。
> 高内聚，低耦合，统一管理，可读性强，增加代码复用性。

###设计模式实现

####1. 工厂模式



##### 1.工厂模式简介

> 工厂模式主要是替换程序中的 new 对象的写法；
>

例如:
```php
$db = new Database($host, $user, $passwd, $port);
```

替换为

```php
Factory::createDatabase();//不需要传参
```

##### 2. 工厂模式好处

> 便于修改，统一管理。
> 假设程序代码里面有几十个 `new Database()`对象写法，程序需要添加或者是删除一个初始化参数的，就需要修改几十个地方，这样的程序修改起来特别不方便**难以维护**，可能会造成**漏改的地方**，还会让程序**难以重构优化**
>
> 使用工厂模式只需要修改一个地方即可
>

```php
class Factory 
{
    static function createDatabase()
    {
		$db = new Database($host, $user, $passwd, $port);
		return $db;
    }
}
```

>- 其他多种设计模式也是依赖于工厂模式;

####2.单例模式

#####1. 单例模式简介

> 单例模式主要是保证一个类仅有一个实例，并提供一个访问它的全局访问方法。
> 

例如：
```php
$db = new Database($host, $user, $passwd, $port);
```
替换为
```php
$db = Database::getInstance();
```
Database.php
```php
class Database 
{
    public function __construct($host, $user, $passwd, $port = 3306)
    {
		$db = new MySQLi ($host, $user, $passwd, $port);
		return $db;
    }
}
```
替换为
```php
class Database 
{
	private static $db;
	
	//construct设置为 private/protected 方法，这样外部就不能对这个类执行 new Database() 操作了
    private function __construct($host, $user, $passwd, $port)
    {
		$db = new MySQLi ($host, $user, $passwd, $port);
		return $db;
    }
    
    public static function getInstance()
    {
        if(self::$db){
            return self::$db;
        }else{
            //$host, $user, $passwd, $port此处未定义
        	self::$db = new self($host, $user, $passwd, $port);
            return self::$db;
        }
    }
    
    
}
```
##### 2. 单例模式好处

> 单例模式只需要实例化一次这个类，减少类的实例化次数，减少程序的复杂度，每次都访问同一对象，程序也不需要分配多个变量来保存实例化对象，减少内存使用。
>
> 假如是数据库连接，也能减少链接次数，优化 IO 操作。



##### 3. 单例模式与工厂模式的结合

Factory.php修改为
```php
class Factory 
{
    static function createDatabase()
    {
		$db = Database::getInstance();
		return $db;
    }
}
```



#### 3. 注册树模式

##### 1. 注册树模式简介

> 工厂模式和单例模式合并起来后，还是会有缺点：
>
> 1. 每次要使用这个类的时候都需要去实例化对象，调用工厂方法或者`getInstence`方法
> 2. 单例模式创建唯一对象的过程本身还有一种判断，即判断对象是否存在。存在则返回对象，不存在则创建对象并返回。 每次创建实例对象都要存在这么一层判断

##### 2. 注册树模式例子

Register.php

```PHP
class Register
{
	//存储实例化对象的数组
    protected static $arrObject;
    
    public static function _set($alias, $object)
    {
        self::$arrObject[$alias] = $object;
    }
    
    public static function _unset($alias, $object)
    {
       unset(self::$arrObject[$alias]);
    }
    
    public static function _get($alias)
    {
         return self::$arrObject[$alias];
    }
}
```

Initialize.php

```PHP
Register::_set("db",Factory::getDatabase())
```

其他地方就可以调用

```PHP
Register::_get("db");
```

##### 3. 注册树模式好处

> 1. 优化了工厂模式和单例模式
> 2. 对象的调用使用静态方法 `Register::_get("db");`，变得统一方便快捷了
> 3. 对象的实例化也统一使用`Register::_set("db",$object);`



#### 4. 适配器模式

##### 1. 适配器模式简介

> 适配器模式主要作用是对不同的函数或者操作方法通过接口类封装成具有统一操作方式的类
>
> 1. 数据库操作有 Mysql，Mysqli，PDO 等，可以把他们封装成统一的操作，当不同的服务器的环境不同的时候，可以简单的通过配置就可以修改程序，让程序支持不同的系统环境。
> 2. 缓存的操作方式也同样有不同的方式，File，redis，memcached，apc。也是可以通过接口对不同的缓存方式进行封装，让他们具有统一的操作方法

##### 2. 适配器模式例子

 IDatabase.php

```PHP
interface IDatabase
{
	public function connect($host, $user, $passwd, $dbname);
    public function query($sql);
    public function close();
}
```

Pdo.php

```PHP
class Pdo implements IDatabase
{
    
}
```

Mysqli.php

```PHP
class Mysqli implements IDatabase
{
    
}
```

##### 3. 适配器模式好处

> 1. 让程序可以快速的兼容不同的系统环境
> 2. 更加统一的操作方式



#### 5. 策略模式

##### 1. 策略模式简介

> 策略模式就是让程序根据不同的环境去选择对应的策略。
>
> 例如：用户性别，用户等级，帖子的热标签，最新标签，去选择对应的策略

##### 2. 策略模式实现方法

`IUserGenderStrategy.php `

```PHP
//根据用户性别制定的策略接口
interface IUserGenderStrategy
{
    //根据不同的性别展示不同的商品类目
	public function showCategory();

    //根据不同的性别展示不同的广告
	public function showAd();
}
```

`FemaleUserGenderStrategy.php`

```PHP
class FemaleUserGenderStrategy extend IUserGenderStrategy
{
    public function showCategory(){
        return "女装类目";
    }
    
    public function showAd(){
        return "女装广告";
    }
}
```

`MaleUserGenderStrategy.php`

```PHP
class MaleUserGenderStrategy extend IUserGenderStrategy
{
    public function showCategory(){
        return "电子游戏";
    }
    
    public function showAd(){
        return "电脑广告";
    }
}
```

调用

`Page.php`

```PHP
class Page
{
    public function index()
    {
        //如果在很多地方都有调用下面这个if else
        //这时候要添加一种else if的话
        //就需要修改很多地方了
        if($_GET['gender'] == 'female'){
           retun  '女装类目'.'女装广告';
        }else{
           retun  '电子游戏'.'电脑广告';
        }
    }
}

$page = new Page();
echo $page->index();
```

修改后调用

`Page.php`

```PHP
class Page
{
    /**
     * @var IUserGenderStrategy
     */
    private $strategy;
    
    public function index()
    {
       return $this->strategy->showCategory() . $this->strategy->showAd();
    }
    
    public function setStrategy(IUserGenderStrategy $strategy)
    {
        $this->strategy = $strategy;
    }
}

$page = new Page();
if($_GET['gender'] == 'female'){
    $strategy = new FemaleUserGenderStrategy();
}else{
	$strategy = new MaleUserGenderStrategy();
}
$page->setStrategy($strategy);
echo $page->index();
```



##### 3. 策略模式好处

> 减少程序的硬编码，`if....else...`或`switch...case...`  让程序更容易修改和维护。





#### 6. 观察者模式

##### 1. 观察者模式简介

> 观察者模式可以让程序逻辑解耦合
>
> 一个事件发生后，需要执行一连串的更新操作，传统的编程模式就是在事件发生后直接加上一连串的更新操作，当更新的逻辑增多后，会让程序变得难以维护。耦合性，侵入性太高了。
>
> 使用观察者模式可以让程序低耦合，非侵入式的维护代码。

##### 2. 观察者模式实现方法

调用

`Index.php`

```PHP
class Index
{
    public function index()
    {
       	echo "事件1发生了";
       	echo "逻辑1";
       	echo "逻辑2";
       	echo "逻辑3";
    }
}

$index = new Index();
$index->index();
```

修改后调用

`IndexEvent.php`

```PHP
class IndexEvent extends EventGenerator
{
    public function index()
    {
       	echo "事件1发生了";
        $this->notify();//通知观察者事件1已经发生了
    }
}

class Observer1 implements Observer
{
    public function update()
    {
       	echo "逻辑1";
    }
}

class Observer2 implements Observer
{
    public function update()
    {
       	echo "逻辑2";
    }
}

$indexEvent = new IndexEvent();
//添加观察者
$indexEvent->addObserver(new Observer1);
$indexEvent->addObserver(new Observer2);
//执行事件
$indexEvent->index();



```

`EventGenerator.php` 

```PHP
//事件发生者
abstract class EventGenerator
{
   	$private $observers = [];
    //添加观察者
    function addObserver(Observer $observer){
        $this->observers[] = $observer;
    }
    //通知
    function notify(){
        foreach($this->observers as $observer){
            $observer->update();
        }
    }
}
```

`Observer.php` 

```PHP
//观察者
interface Observer
{
   function update();//更新
}

$index = new Index();
echo $index->index();

```

#### 7. 原型模式

##### 1. 原型模式简介

> 原型模式是先new好一个对象作为原型，然后通过clone原型来创建新的对象

##### 2. 原型模式实现方法

调用

`Canvas.php`

```PHP
class Canvas
{
    public $data;
    
    public function init($width = 20, $height = 10)
    {
       	$data = array();
        for($i = 0; $i < $height; $i++)
        {
            for($j = 0; $j < $width; $j++)
            {
                $data[$i][$j] = '*';
			}
        }
        $this>data = $data;
    }
    
    public function draw(){
        foreach($this->data as $line)
        {
            foreach($line as $char)
            {
                echo $char;
            }
            echo "<br /> \n";
        }
    }
    
    public function rect($a1, $a2, $b1, $b2){
        foreach($this->data as $k1 => $line)
        {
            if($k1 < $a1 || $k1 > $a2) continue;
            foreach($this->data as $k2 => $char)
            {
                if($k2 < $b1 || $k2 > $b2) continue;
				$this->data[$k1][$k2] = '&nbsp;'
            }
        }
    }
    
}

```

`Index.php`

```PHP
//画一个
$canvas1 = new Canvas()
$canvas1->init();
$canvas1->rect(1,3,2,6);
$canvas1->draw();
//在画一个
$canvas2 = new Canvas()
$canvas2->init();
$canvas2->rect(3,6,7,10);
$canvas2->draw();

```

修改后调用

`Index.php`

```PHP
$protoType = new Canvas()
$protoType->init();
//画一个
$canvas1 = clone $protoType;
$canvas1->rect(1,3,2,6);
$canvas1->draw();
//在画一个
$canvas2 = clone $protoType;
$canvas2->rect(3,6,7,10);
$canvas2->draw();


```

##### 3. 原型模式好处

> 原型模式可以优化程序代码。
>
> 避免类创建时的重复初始化操作。如果创建一个对象需要很大的开销，每次都new的话就会开销很大，而原型clone模式只需要内存的拷贝即可。
>
> 上面例子中，每次new一个对象，进行init操作，就需要进行一次20*10 = 两百次的循环。使用原型模式只做了一次内存拷贝。

#### 8. 装饰器模式

##### 1. 装饰器模式简介

> 装饰器模式可以动态的添加修改类的功能。
>
> 一个类提供了一项功能，如果要在修改并添加额外的功能，传统的编程方式就是要写一个子类继承它，并重新实现类的方法
>
> 使用装饰器模式，仅需要在运行时添加一个装饰器对象即可实现，可以实现最大的灵活性

##### 2. 装饰器模式实现方法

> 回到原型模式的例子。
>
> draw()方法，如果现在要修改draw方法，为画布增加颜色，传统的调用方法如下

`Canvas2.php`

```PHP
class Canvas2 extends Canvas
{
    public function draw(){
        echo "<div style='color: red'>";
        parent::draw();
        echo "</div>";
    }
}

```

> draw()方法，如果现在要修改draw方法，修改画布的字体大小，传统的调用方法如下

`Canvas3.php`

```PHP
class Canvas3 extends Canvas
{
    public function draw(){
        echo "<div style='font-size: 20px'>";
        parent::draw();
        echo "</div>";
    }
}

```

> draw()方法，如果现在要修改draw方法，修改画布的颜色和字体大小，传统的实现方式就又要写一个子类继承它，并重写draw方法。
>
> 如果我们有些地方要修改颜色，有些地方要修改字体大小，有些地方要修改颜色和字体大小，那么传统的实现方法写起来就很麻烦，不能灵活的调用。

修改后调用

`Canvas.php`

```PHP
class Canvas
{
    public $data;
    protected $decrators;
    
    public function init($width = 20, $height = 10)
    {
       	$data = array();
        for($i = 0; $i < $height; $i++)
        {
            for($j = 0; $j < $width; $j++)
            {
                $data[$i][$j] = '*';
			}
        }
        $this>data = $data;
    }
    
    public function draw(){
        $this->beforeDraw();//前置操作
        foreach($this->data as $line)
        {
            foreach($line as $char)
            {
                echo $char;
            }
            echo "<br /> \n";
        }
        $this->afterDraw();//后置操作
        //也可以在中间插入操作，实现更多功能
    }
    
    //动态添加装饰方法
    //它接受了一个画布装修器接口
    public function addDecrator(DrawDecrator $decrator){
        $this->decrators[] = $decrator;
    }
    
    //前置操作
    public function beforeDraw(){
        foreach($this->decrators as $decrator){
            $decrator->beforeDraw();
        }
    }
    
    //后置操作
    public function afterDraw(){
    	//这里需要反转一下装饰器数组
        $decrators = array_reverse($this->decrators);
        foreach($decrators as $decrator){
        	$decrator->afterDraw();
        }
    }
    
    public function rect($a1, $a2, $b1, $b2){
        foreach($this->data as $k1 => $line)
        {
            if($k1 < $a1 || $k1 > $a2) continue;
            foreach($this->data as $k2 => $char)
            {
                if($k2 < $b1 || $k2 > $b2) continue;
				$this->data[$k1][$k2] = '&nbsp;'
            }
        }
    }
    
}


```

`DrawDecrator.php`  装饰器接口

```PHP
interface DrawDecrator
{
    //前置操作
    public function beforeDraw();
    
    //后置操作
    public function afterDraw();
    
}


```

`ColorDrawDecrator.php` 颜色装饰器

```PHP
class ColorDrawDecrator implements DrawDecrator
{
    //前置操作
    public function beforeDraw(){
        echo "<div style='color: red'>";
    }
    
    //后置操作
    public function afterDraw(){
        echo "</div>";
    }
}


```

`FontDrawDecrator.php` 字体装饰器

```PHP
class FontDrawDecrator implements DrawDecrator
{
    //前置操作
    public function beforeDraw(){
        echo "<div style='font-size: 20px;'>";
    }
    
    //后置操作
    public function afterDraw(){
        echo "</div>";
    }
}


```

`Index.php`

```PHP
$protoType = new Canvas()
$protoType->init();
//画一个
$canvas1 = clone $protoType;
$canvas1->addDecrator( new ColorDrawDecrator());
$canvas1->addDecrator( new FontDrawDecrator());
$canvas1->rect(1,3,2,6);
$canvas1->draw();


```

##### 3. 装饰器模式好处

> 装饰器模式让程序变得更加灵活，方便修改程序。

#### 9. 迭代器模式

##### 1. 迭代器模式简介

> 怎么理解迭代器模式？
>
> 第一，要知道迭代器的内部通用方法（current,key,next,rewind,valid），迭代器提供了标准的遍历对象的方法。
>
> 第二，知道迭代器的返回结果，迭代器模式实现了迭代器接口，返回的是一个迭代器对象。就是说只要你一个对象实现了迭代器模式，你要怎么去遍历都可以。
>
> 第三，对象的私用属性也能遍历出来。
>
> 第四，  PHP提供了几十种的标准迭代器（SPL），以便以遍历不同的对象。
>
> 1，添加迭代器
>
> 2，数组迭代器
>
> 3，缓存迭代器
>
> 4，回调过滤器迭代器
>
> 5，目录迭代器
>
> 6，空迭代器
>
> 7，文件系统迭代器
>
> 8，过滤器迭代器
>
> 9，全局迭代器
>
> 10，无限的迭代器
>
> 11，迭代器迭代器
>
> 12，限制迭代器
>
> 13，多个迭代器
>
> 14，没有重置迭代器
>
> 15，腹肌迭代器
>
> 16，递归迭代器阵列
>
> 17，递归迭代器缓存
>
> 18，递归回调筛选器迭代器
>
> 19，递归目录迭代器
>
> 20，递归滤波器迭代器
>
> 21，递归迭代器迭代器
>
> 22，递归正则迭代器
>
> 23，递归树迭代器
>
> 24，正则迭代器
>
> 白话解释：只要你的类实现（`implements`）了迭代器`iterator`接口`interface`，那你就能使用for,foreach,while的任何一种方法去遍历它。
>
> 
>
> 
>
> 迭代器的使用范围？
>
> 假设如以下这个东西：
>
> ```
> $data = [1,2,3];
> foreach($data as $val)
> {
> echo $val;
> }
> 
> ```
>
> 我们不用foreach的话，要输出里面的每个子数据只能是这样：                    
>
> ```
> echo $data[0];
> echo $data[1];
> echo $data[2];
> 
> ```
>
> foreach只是一个循环遍历的语句结构，可以减少我们的程序代码，让程序自动执行重复的逻辑结构。
>
> 然后，回过头来说迭代器，先来看整个迭代器的结构：
>
> ```
> Iterator extends Traversable {
> /* 方法 */
> abstract public mixed current ( void )
> abstract public scalar key ( void )
> abstract public void next ( void )
> abstract public void rewind ( void )
> abstract public boolean valid ( void )
> }
> 
> ```
>
> 很清晰的看到，迭代器提供的是一整套操作子数据的接口，foreach也就每次可以通过next移动指针（当然，迭代器的指针具体移动需要我们自己实现），然后通过valid检查迭代是否可结束。
>
> 那么，这样子一个东西有什么作用呢？
>
> 1. 可以处理一个庞大数据量的问题，减少内存，这个东西可以看这样子一套数据，假设有1亿条数据的数组，按传统的遍历，我需要把这套东西放到内存，然后用foreach遍历，但是如果是迭代器的话，我每次迭代的时候，移动一下指针，去硬盘上取这么一条数据就可以了。
> 2. 另外一个就是一些特殊数据的遍历了。比如我要遍历一个对象，就特别有用。
> 3. 还有就是有些数据需要手动进行遍历，不需要通过foreach一次性处理完的时候，也很有用。
>
> 
>
> 哦，然后说一下，这个迭代器和foreach压根是两个不同的东西，foreach是提供对迭代器进行遍历的一个工具，而迭代器的遍历不一定需要依赖于foreach遍历。
> 对迭代器的遍历也可以通过while来遍历的，如：
>
> ```
> while($iterator->valid()) 
> {
> 	$iterator->next();
> }
> 
> ```
>
> 特别说明一下对对象的遍历，一般人觉得所谓的遍历对象就是对一个对象里的属性或者方法一个一个的取出来，然后做输出或者处理，实际上，这里的迭代器对对象的遍历并不是这个意思，可能这句话本身的描述有问题。

###参考文献

[大话PHP设计模式]: https://www.imooc.com/learn/236	"作者 rango"







