# 设计模式

## 创建型模式

创建型模式(Creational Pattern)对类的实例化过程进行了抽象，能够将软件模块中对象的创建和对象的使用分离。为了使软件的结构更加清晰，外界对于这些对象只需要知道它们共同的接口，而不清楚其具体的实现细节，使整个系统的设计更加符合单一职责原则。

创建型模式在创建什么(What)，由谁创建(Who)，何时创建(When)等方面都为软件设计者提供了尽可能大的灵活性。创建型模式隐藏了类的实例的创建细节，通过隐藏对象如何被创建和组合在一起达到使整个系统独立的目的。

### 简单工厂模式( Simple Factory Pattern )

简单工厂模式(Simple Factory Pattern)：又称为静态工厂方法(Static Factory Method)模式。简单工厂模式专门定义一个类来负责创建其他类的实例，可以根据参数的不同返回不同类的实例，被创建的实例通常都具有共同的父类。

简单工厂模式包含如下角色：
- Factory：工厂角色负责实现创建所有实例的内部逻辑。
- Product：抽象产品角色是所创建的所有对象的父类，负责描述所有实例所共有的公共接口。
- ConcreteProduct：具体产品角色是创建目标，所有创建的对象都充当这个角色的某个具体类的实例。

使用简单工厂模式可以将产品的“消费”和“生产”完全分开，客户端只需要知道自己需要什么产品，如何来使用产品就可以了，具体的产品生产任务由具体的工厂类来实现。工厂类根据传进来的参数生产具体的产品供消费者使用。这种模式使得更加利于扩展，当有新的产品加入时仅仅需要在工厂中加入新产品的构造就可以了。

简单工厂模式最大的问题在于工厂类的职责相对过重，增加新的产品需要修改工厂类的判断逻辑，这一点与开闭原则是相违背的。

```php
//创建抽象产品类“人”
abstract class Person
{
    abstract public function say();
}
//创建具体产品类（继承抽象产品类）：“男人”、“女孩”
class Man extends Person
{
    public function say()
    {
        echo 'I am a strong man!';
    }
}
class Girl extends Person
{
    public function say()
    {
        echo 'I am a smarty girl!';
    }
}

//创建工厂类，通过创建静态方法从而根据传入不同参数创建不同具体实现类的实例
class Factory
{
    public static function createPerson($person)
    {
        switch ($person) {
            case 'girl':
                return new Girl();
                break;
            case 'man':
                return new Man();
                break;
            default:
                return new Man();
                break;
        }
    }
}
Factory::createPerson('girl')->say(); //I am a smarty girl!
```

### 工厂方法模式(Factory Method Pattern)

工厂方法模式(Factory Method Pattern)又称为工厂模式，也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式。

在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作下推到工厂子类中完成，即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

工厂方法模式包含如下角色：
- Product：抽象产品
- ConcreteProduct：具体产品
- Factory：抽象工厂
- ConcreteFactory：具体工厂

工厂方法模式是简单工厂模式的进一步抽象和推广。由于使用了面向对象的多态性，工厂方法模式保持了简单工厂模式的优点，而且克服了它的缺点。

在工厂方法模式中，核心的工厂类不再负责所有产品的创建，而是将具体创建工作交给子类去做。核心类仅仅负责给出具体工厂必须实现的接口，而不负责哪一个产品类被实例化这种细节，这使得工厂方法模式可以允许系统在不修改工厂角色的情况下引进新产品，更加符合“开闭原则”。

```php
//产品类（Person及其子类）不做变更

//工厂类更新为 抽象类（接口） + 两个实现
interface Factory
{
    public static function createPerson();
}
class GirlFactory implements Factory
{
    public static function createPerson()
    {
        return new Girl();
    }
}
class ManFactory implements Factory
{
    public static function createPerson()
    {
        return new Man();
    }
}
ManFactory::createPerson()->say();
```

### 抽象工厂模式(Abstract Factory)
抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式。

在工厂方法模式中具体工厂负责生产具体的产品，每一个具体工厂对应一种具体产品，工厂方法也具有唯一性，一般情况下，一个具体工厂中只有一个工厂方法或者一组重载的工厂方法。但是有时候我们需要一个工厂可以提供多个产品对象，而不是单一的产品对象。

为了更清晰地理解工厂方法模式，需要先引入两个概念：
- 产品等级结构：产品等级结构即产品的继承结构，如一个抽象类是电视机（男人），其子类有海尔电视机（黑男人）、海信电视机（白男人）、TCL电视机（黄男人），则抽象电视机（男人）与具体品牌（肤色）的电视机（男人）之间构成了一个产品等级结构，抽象电视机（男人）是父类，而具体品牌（肤色）的电视机（男人）是其子类。
- 产品族 ：在抽象工厂模式中，产品族是指由同一个工厂生产的，位于不同产品等级结构中的一组产品，如海尔电器工厂（黑色任重）生产的海尔电视机（黑男人）、海尔电冰箱（黑女孩），海尔电视机（黑男人）位于电视机（男人）产品等级结构中，海尔电冰箱（黑女孩儿）位于电冰箱（女孩儿）产品等级结构中。

当系统所提供的工厂所需生产的具体产品并不是一个简单的对象，而是多个位于不同产品等级结构中属于不同类型的具体产品时需要使用抽象工厂模式。

抽象工厂模式是所有形式的工厂模式中最为抽象和最具一般性的一种形态。

抽象工厂模式与工厂方法模式最大的区别在于，工厂方法模式针对的是一个产品等级结构，而抽象工厂模式则需要面对多个产品等级结构，一个工厂等级结构可以负责多个不同产品等级结构中的产品对象的创建 。当一个工厂等级结构可以创建出分属于不同产品等级结构的一个产品族中的所有对象时，抽象工厂模式比工厂方法模式更为简单、有效率。

抽象工厂模式包含如下角色：
- Factory：抽象工厂
- ConcreteFactory：具体工厂
- AbstractProduct：抽象产品
- Product：具体产品

```php
//人
interface Person
{
    public function say();
}
interface Man extends Person
{
}
interface Girl extends Person
{
}

class BlackMan implements Man
{
    public function say()
    {
        echo 'I am a black man!' . PHP_EOL;
    }
}
class WhiteMan implements Man
{
    public function say()
    {
        echo 'I am a white man!' . PHP_EOL;
    }
}
class BlackGirl implements Girl
{
    public function say()
    {
        echo 'I am a black girl!' . PHP_EOL;
    }
}
class WhiteGirl implements Girl
{
    public function say()
    {
        echo 'I am a white girl!' . PHP_EOL;
    }
}

//工厂
interface Factory
{
    public function createWhite();

    public function createBlack();
}
class ManFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteMan();
    }

    public function createBlack()
    {
        return new BlackMan();
    }
}
class GirlFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteGirl();
    }

    public function createBlack()
    {
        return new BlackGirl();
    }
}

//使用
$manFactory = new ManFactory();
$manFactory->createWhite()->say();
$manFactory->createBlack()->say();

$girlFactory = new GirlFactory();
$girlFactory->createWhite()->say();
$girlFactory->createBlack()->say();
```

如果此时需要增加增加一个“男孩儿”产品等级，只需要增加
```php
interface Boy extends Person
{
}
class BlackBoy implements Man
{
    public function say()
    {
        echo 'I am a black boy!' . PHP_EOL;
    }
}
class WhiteBoy implements Man
{
    public function say()
    {
        echo 'I am a white boy!' . PHP_EOL;
    }
}

class BoyFactory implements Factory
{
    public function createWhite()
    {
        return new WhiteBoy();
    }

    public function createBlack()
    {
        return new BlackBoy();
    }
}
```

如果此时需要增加一个肤色 - 黄种人，需要增加实现
```php
class YellowMan implements Man
{
    public function say()
    {
        echo 'I am a yellow man!' . PHP_EOL;
    }
}

class YellowGirl implements Girl
{
    public function say()
    {
        echo 'I am a yellow girl!' . PHP_EOL;
    }
}

class YellowBoy implements Boy
{
    public function say()
    {
        echo 'I am a yellow boy!' . PHP_EOL;
    }
}

interface Factory
{
    //...
    // 工厂接口增加 createYellow 方法约束
    public function createYellow();
    //...
}

class ManFactory implements Factory
{
    //...
    // 工厂 ManFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowMan();
    }
    //...
}

class GilrFactory implements Factory
{
    //...
    // 工厂 GilrFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowGilr();
    }
    //...
}

class BoyFactory implements Factory
{
    //...
    // 工厂 BoyFactory 增加 createYellow 实现
    public function createYellow()
    {
        return new YellowBoy();
    }
    //...
}
```

参考：https://www.zhihu.com/question/20367734

### 建造者模式（Builder Pattern）
建造者模式(Builder Pattern)：又称生成器模式，将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示。

建造者模式是一步一步创建一个包含多个组成部件的复杂对象，它允许用户只通过指定复杂对象的类型和内容就可以构建它们，用户不需要知道内部的具体构建细节，相同的构建过程可以创建不同的产品。

复杂对象相当于一辆有待建造的汽车，而对象的属性相当于汽车的部件，建造产品的过程就相当于组合部件的过程。

对大多数用户而言，无须知道这些部件的装配细节，也几乎不会使用单独某个部件，而是使用一辆完整的汽车，可以通过建造者模式对其进行设计与描述，建造者模式可以将部件和其组装过程分开，一步一步创建一个复杂的对象。用户只需要指定复杂对象的类型就可以得到该对象，而无须知道其内部的具体构造细节。

由于组合部件的过程很复杂，因此，这些部件的组合过程往往被“外部化”到一个称作建造者的对象里，建造者返还给客户端的是一个已经建造完毕的完整产品对象，而用户无须关心该对象所包含的属性以及它们的组装方式，这就是建造者模式的模式动机。

在软件开发中，也存在大量类似汽车一样的复杂对象，它们拥有一系列成员属性，这些成员属性中有些是引用类型的成员对象。而且在这些复杂对象中，还可能存在一些限制条件，如某些属性没有赋值则复杂对象不能作为一个完整的产品使用；有些属性的赋值必须按照某个顺序，一个属性没有赋值之前，另一个属性可能无法赋值等。

建造者模式包含如下角色：
- Builder：抽象建造者，将建造的具体过程交与它的子类来实现，更容易扩展。一般至少会有两个抽象方法，一个用来建造产品，一个是用来返回产品。
- ConcreteBuilder：具体建造者。
- Director：指挥者，负责调用适当的建造者来组建产品，一般不与产品类发生依赖关系，与建造者类接交互，一般指挥者被用来封装程序中易变的部分。
- Product：产品角色，一般是一个创建过程较为复杂的对象，一般会有比较多的代码量。实际编程中，产品类可以是由一个抽象类与它的不同实现组成，也可以是由多个抽象类与他们的实现组成。

```php
//产品角色
class Product
{
    private $name;
    private $type;

    public function showProduct()
    {
        echo "名称：" . $this->name . PHP_EOL;
        echo "型号：" . $this->type . PHP_EOL;
    }

    public function setName($name)
    {
        $this->name = $name;
    }

    public function setType($type)
    {
        $this->type = $type;
    }
}
//抽象建造者
interface Builder
{
    public function setPart($name, $type);

    public function getProduct();
}
//具体建造者
class ConcreteBuilder implements Builder
{
    private $product = null;

    public function __construct()
    {
        $this->product = new Product();
    }

    public function getProduct()
    {
        return $this->product;
    }

    public function setPart($name, $type)
    {
        $this->product->setName($name);
        $this->product->setType($type);
    }
}
//指挥者
class Director
{
    private $builder = null;
    public function __construct()
    {
        $this->builder = new ConcreteBuilder();
    }

    public function getAudi()
    {
        $this->builder->setPart("奥迪汽车", "Q5");
        return $this->builder->getProduct();
    }

    public function getBmw()
    {
        $this->builder->setPart("宝马汽车", "X7");
        return $this->builder->getProduct();
    }
}

$director = new Director();

$audi = $director->getAudi();
$audi->showProduct();

$bmw = $director->getBmw();
$bmw->showProduct();
```

### 单例模式
单例模式(Singleton Pattern)：单例模式确保某一个类只有一个实例，而且自行实例化并向整个系统提供这个实例，这个类称为单例类，它提供全局访问的方法。

单例模式的要点有三个：一是某个类只能有一个实例；二是它必须自行创建这个实例；三是它必须自行向整个系统提供这个实例。

单例模式包含如下角色：
- Singleton：单例

单例模式的目的是保证一个类仅有一个实例，并提供一个访问它的全局访问点。单例模式包含的角色只有一个，就是单例类——Singleton。

单例类拥有一个私有构造函数，确保用户无法通过new关键字直接实例化它。除此之外，该模式中包含一个静态私有成员变量与静态公有的工厂方法，该工厂方法负责检验实例的存在性并实例化自己，然后存储在静态成员变量中，以确保只有一个实例被创建。

在单例模式的实现过程中，需要注意如下三点：
- 单例类的构造函数为私有；
- 提供一个自身的静态私有成员变量；
- 提供一个公有的静态工厂方法。

```php

class DB
{
    private static $instance = null;

    private function __construct()
    {
    }

    private function __clone()
    {
    }

    public static function getInstance()
    {
        if (self::$instance == null) {
            self::$instance = new self();
        }
        return self::$instance;
    }
}
```

## 结构型模式
结构型模式(Structural Pattern)描述如何将类或者对象结合在一起形成更大的结构，就像搭积木，可以通过 简单积木的组合形成复杂的、功能更为强大的结构。

结构型模式可以分为类结构型模式和对象结构型模式：
- 类结构型模式关心类的组合，由多个类可以组合成一个更大的系统，在类结构型模式中一般只存在继承关系和实现关系。 
- 对象结构型模式关心类与对象的组合，通过关联关系使得在一个类中定义另一个类的实例对象，然后通过该对象调用其方法。

根据“合成复用原则”，在系统中尽量使用关联关系来替代继承关系，因此大部分结构型模式都是对象结构型模式。

### 适配器模式(Adapter Pattern) 
适配器模式(Adapter Pattern)：将一个接口转换成客户希望的另一个接口，适配器模式使接口不兼容的那些类可以一起工作，其别名为包装器(Wrapper)。适配器模式既可以作为类结构型模式，也可以作为对象结构型模式。

在软件开发中采用类似于电源适配器的设计和编码技巧被称为适配器模式。

通常情况下，客户端可以通过目标类的接口访问它所提供的服务。有时，现有的类可以满足客户类的功能需要，但是它所提供的接口不一定是客户类所期望的，这可能是因为现有类中方法名与目标类中定义的方法名不一致等原因所导致的。

在这种情况下，现有的接口需要转化为客户类期望的接口，这样保证了对现有类的重用。如果不进行这样的转化，客户类就不能利用现有类所提供的功能，适配器模式可以完成这样的转化。

在适配器模式中可以定义一个包装类，包装不兼容接口的对象，这个包装类指的就是适配器(Adapter)，它所包装的对象就是适配者(Adaptee)，即被适配的类。

适配器提供客户类需要的接口，适配器的实现就是把客户类的请求转化为对适配者的相应接口的调用。也就是说：当客户类调用适配器的方法时，在适配器类的内部将调用适配者类的方法，而这个过程对客户类是透明的，客户类并不直接访问适配者类。因此，适配器可以使由于接口不兼容而不能交互的类可以一起工作。这就是适配器模式的模式动机。

适配器模式包含如下角色：
- Target：目标抽象类
- Adapter：适配器类
- Adaptee：适配者类
- Client：客户类

假设在`Client`（项目）中一直使用的`find`作为查询数据库的接口。此时引入了一个新的数据库操作类`Adaptee`，此类提供的查询接口为`select`。那么不能为了此类修改所有客户端代码，也不可能为了这一个类而修改其他类的查询接口。

此时引入适配器`Adapter`，对客户端提供的`find`方法中执行适配者类`Adaptee`的`select`方法。
```php
//适配者类
class Adaptee
{
    public function select($where)
    {
        echo "This is David";
    }
}

//目标抽象类
interface Target
{
    public function find($where);
}

//适配器类
class Adapter implements Target
{
    private $instance = null;
    public function __construct()
    {
        $this->instance = new Adaptee();
    }

    public function find($where)
    {
        $this->instance->select($where);
    }
}

//客户类
class Client
{
    public function __construct()
    {
        $mysql = new Adapter();
        $mysql->find('students');
    }
}
```

### 桥接模式(Bridge Pattern)
桥接模式(Bridge Pattern)：将抽象部分与它的实现部分分离，使它们都可以独立地变化。它是一种对象结构型模式，又称为柄体(Handle and Body)模式或接口(Interface)模式。

假设要绘制矩形、圆形、椭圆、正方形，我们至少需要4个形状类。如果绘制的图形要具有不同的颜色，如红色、绿色、蓝色等，此时至少有如下两种设计方案：
- 第一种设计方案是为每一种形状都提供一套各种颜色的版本。
- 第二种设计方案是根据实际需要对形状和颜色进行组合

对于有两个变化维度（即两个变化的原因）的系统，采用方案二来进行设计系统中类的个数更少，且系统扩展更为方便。设计方案二即是桥接模式的应用。

桥接模式主要应对的是由于实际的需要，某个类具有两个或者两个以上的维度变化（违反了SRP原则），如果只是用继承将无法实现这种需要，或者使得设计变得相当臃肿。

桥接模式将继承关系转换为关联关系，从而降低了类与类之间的耦合，减少了代码编写量。

桥接模式包含如下角色：
- Abstraction：定义抽象接口，拥有一个Implementor类型的对象引用
- RefinedAbstraction：扩展Abstraction中的接口定义
- Implementor：具体实现的接口，Implementor和RefinedAbstraction接口并不一定完全一致，实际上这两个接口可以完全不一样Implementor提供具体操作方法，而Abstraction提供更高层次的调用
- ConcreteImplementor：具体实现类，实现Implementor接口

```php
// 形状接口 Implementor
interface Shape
{
    public function Draw();
}

// 具体形状 ConcreteImplementor
class Circular implements Shape
{
    public function Draw()
    {
        echo '圆形';
    }
}
class Square implements Shape
{
    public function Draw()
    {
        echo '正方形';
    }
}

// 颜色的抽象接口 Abstraction
abstract class Color
{
    /**
     * @var Shape
     */
    protected $shape;

    public function __construct(Shape $shape)
    {
        $this->shape = new $shape;
    }

    abstract public function Draw();
}

//具体的颜色 RefinedAbstraction
class Red extends Color
{
    public function Draw()
    {
        echo '红色的';
        $this->shape->Draw();
    }
}
class Blue extends Color
{
    public function Draw()
    {
        echo '蓝色的';
        $this->shape->Draw();
    }
}

(new Blue(new Circular()))->Draw();echo PHP_EOL;
(new Red(new Square()))->Draw();
```

### 装饰模式(Decorator Pattern)
装饰模式(Decorator Pattern)：动态地给一个对象增加一些额外的职责(Responsibility)，就增加对象功能来说，装饰模式比生成子类实现更为灵活。其别名也可以称为包装器(Wrapper)，与适配器模式的别名相同，但它们适用于不同的场合。根据翻译的不同，装饰模式也有人称之为“油漆工模式”。

一般有两种方式可以实现给一个类或对象增加行为：
- 继承机制，使用继承机制是给现有类添加功能的一种有效途径，通过继承一个现有类可以使得子类在拥有自身方法的同时还拥有父类的方法。但是这种方法是静态的，用户不能控制增加行为的方式和时机。
- 关联机制，即将一个类的对象嵌入另一个对象中，由另一个对象来决定是否调用嵌入对象的行为以便扩展自己的行为，我们称这个嵌入的对象为装饰器(Decorator)

装饰模式以对客户透明的方式动态地给一个对象附加上更多的责任，换言之，客户端并不会觉得对象在装饰前和装饰后有什么不同。装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。

装饰模式包含如下角色：
- Component: 抽象构件，装饰类和具体构件的公共父类，定义一个抽象接口以规范准备接收附加责任的对象。
- ConcreteComponent: 实现抽象构件，通过装饰角色为其添加一些职责。
- Decorator: 抽象装饰类，继承抽象构件，并包含具体构件的实例，可以通过其子类扩展具体构件的功能。
- ConcreteDecorator: 具体装饰类，实现抽象装饰的相关方法，并给具体构件对象添加附加的责任。

```php
//抽象构件
interface Component
{
    public function say();
}

//具体构件
class ConcreteComponent implements Component
{
    public function say()
    {
        echo '我是具体构件' . PHP_EOL;
    }
}

//抽象装饰
class Decorator implements Component
{
    /**
     * @var Component
     */
    protected $component = null;

    public function __construct(Component $component)
    {
        $this->component = $component;
    }

    public function say()
    {
        $this->component->say();
    }
}

//具体装饰者
class ConcreteDecorator extends Decorator
{
    public function say()
    {
        parent::say();
        $this->addedFunction();
    }

    private function addedFunction()
    {
        echo "我做了点装饰" . PHP_EOL;
    }
}

$component = new ConcreteComponent();
$component->say();

$decorator = new ConcreteDecorator($component);
$decorator->say();
```
### 外观模式(Facade Pattern)
外观模式(Facade Pattern)：外部与一个子系统的通信必须通过一个统一的外观对象进行，为子系统中的一组接口提供一个一致的界面，外观模式定义了一个高层接口，这个接口使得这一子系统更加容易使用。外观模式又称为门面模式，它是一种对象结构型模式。

外观模式包含如下角色：
- Facade: 外观角色
- SubSystem:子系统角色
```php
//Facade
class Facade
{
    private $subSystemA = null;
    private $subSystemB = null;
    private $subSystemC = null;

    public function __construct()
    {
        $this->subSystemA = new SubSystemA();
        $this->subSystemB = new SubSystemB();
        $this->subSystemC = new SubSystemC();
    }

    public function operation()
    {
        $this->subSystemA->operation();
        $this->subSystemB->operation();
        $this->subSystemC->operation();
    }
}

//SubSystem
class SubSystemA
{
    public function operation()
    {
        echo "SubSystemA" . PHP_EOL;
    }
}

class SubSystemB
{
    public function operation()
    {
        echo "SubSystemB" . PHP_EOL;
    }
}

class SubSystemC
{
    public function operation()
    {
        echo "SubSystemC" . PHP_EOL;
    }
}

(new Facade())->operation();
```

### 享元模式(Flyweight Pattern)
享元模式(Flyweight Pattern)：运用共享技术有效地支持大量细粒度对象的复用。系统只使用少量的对象，而这些对象都很相似，状态变化很小，可以实现对象的多次复用。由于享元模式要求能够共享的对象必须是细粒度对象，因此它又称为轻量级模式，享元模式结构较为复杂，一般结合工厂模式一起使用。

享元模式通过共享技术实现相同或相似对象的重用。

在享元模式中可以共享的相同内容称为内部状态(IntrinsicState)，而那些需要外部环境来设置的不能共享的内容称为外部状态(Extrinsic State)，由于区分了内部状态和外部状态，因此可以通过设置不同的外部状态使得相同的对象可以具有一些不同的特征，而相同的内部状态是可以共享的。

在享元模式中通常会出现工厂模式，需要创建一个享元工厂来负责维护一个享元池(Flyweight Pool)用于存储具有相同内部状态的享元对象。

在享元模式中共享的是享元对象的内部状态，外部状态需要通过环境来设置。在实际使用中，能够共享的内部状态是有限的，因此享元对象一般都设计为较小的对象，它所包含的内部状态较少，这种对象也称为细粒度对象。享元模式的目的就是使用共享技术来实现大量细粒度对象的复用。

享元模式包含如下角色：
- Flyweight: 抽象享元类，通常是一个接口或抽象类，在抽象享元类中声明了具体享元类公共的方法，这些方法可以向外界提供享元对象的内部数据（内部状态），同时也可以通过这些方法来设置外部数据（外部状态）。
- ConcreteFlyweight: 具体享元类，实现了抽象享元类，其实例称为享元对象；在具体享元类中为内部状态提供了存储空间。通常我们可以结合单例模式来设计具体享元类，为每一个具体享元类提供唯一的享元对象。
- UnsharedConcreteFlyweight: 非共享具体享元类，并不是所有的抽象享元类的子类都需要被共享，不能被共享的子类可设计为非共享具体享元类；当需要一个非共享具体享元类的对象时可以直接通过实例化创建。
- FlyweightFactory: 享元工厂类，享元工厂类用于创建并管理享元对象，它针对抽象享元类编程，将各种类型的具体享元对象存储在一个享元池中，享元池一般设计为一个存储“键值对”的集合（也可以是其他类型的集合），可以结合工厂模式进行设计；当用户请求一个具体享元对象时，享元工厂提供一个存储在享元池中已创建的实例或者创建一个新的实例（如果不存在的话），返回新创建的实例并将其存储在享元池中。

```php
// Flyweight: 抽象享元类
interface Flyweight
{
    public function operation($name);

    public function getType();
}
// ConcreteFlyweight: 具体享元类
class ConcreteFlyweight implements Flyweight
{
    private $type;

    public function __construct($type)
    {
        $this->type = $type;
    }

    public function operation($name)
    {
        echo "[类型(内在状态)] - [{$this->type}] - [名字(外在状态)] - [{$name}]".PHP_EOL;
    }

    public function getType()
    {
        return $this->type;
    }
}
//FlyweightFactory: 享元工厂类
class FlyweightFactory
{
    private static $instances = [];

    /**
     * @param $type
     * @return ConcreteFlyweight
     */
    public static function getShape($type)
    {
        if (!isset(self::$instances[$type]) || is_null(self::$instances[$type])) {
            self::$instances[$type] = new ConcreteFlyweight($type);
        }

        return self::$instances[$type];
    }

    public static function total()
    {
        return count(self::$instances);
    }
}

$fw1 = FlyweightFactory::getShape('Iphone');
$fw2 = FlyweightFactory::getShape('HuaWei');
$fw1->operation('7');
$fw2->operation("P30");
```

### 代理模式(Proxy Pattern)
代理模式(Proxy Pattern)：给某一个对象提供一个代 理，并由代理对象控制对原对象的引用。代理模式的英 文叫做Proxy或Surrogate，它是一种对象结构型模式。

在某些情况下，一个客户不想或者不能直接引用一个对象，此时可以通过一个称之为“代理”的第三者来实现间接引用。代理对象可以在客户端和目标对象之间起到中介的作用，并且可以通过代理对象去掉客户不能看到的内容和服务或者添加客户需要的额外服务。

通过引入一个新的对象（如小图片和远程代理对象）来实现对真实对象的操作或者将新的对象作为真实对象的一个替身，这种实现机制即为代理模式，通过引入代理对象来间接访问一 个对象，这就是代理模式的模式动机。

代理模式包含如下角色：
- Subject: 抽象主题角色
- Proxy: 代理主题角色
- RealSubject: 真实主题角色
```php
interface Subject
{
    public function operation();
}

class RealSubject implements Subject
{
    public function operation()
    {
        echo '我是真的执行了'.PHP_EOL;
    }
}

class Proxy implements Subject
{
    private $subject;

    public function __construct(Subject $subject)
    {
        $this->subject = $subject;
    }

    public function operation()
    {
        echo '经过代理增强后 ';
        $this->subject->operation();
    }
}

$subject = new RealSubject();
$proxy = new Proxy($subject);
$proxy->operation();
```

## 行为型模式
行为型模式(Behavioral Pattern)是对在不同的对象之间划分责任和算法的抽象化。

行为型模式不仅仅关注类和对象的结构，而且重点关注它们之间的相互作用。

通过行为型模式，可以更加清晰地划分类与对象的职责，并研究系统在运行时实例对象 之间的交互。在系统运行时，对象并不是孤立的，它们可以通过相互通信与协作完成某些复杂功能，一个对象在运行时也将影响到其他对象的运行。

行为型模式分为类行为型模式和对象行为型模式两种：

- 类行为型模式：类的行为型模式使用继承关系在几个类之间分配行为，类行为型模式主要通过多态等方式来分配父类与子类的职责。
- 对象行为型模式：对象的行为型模式则使用对象的聚合关联关系来分配行为，对象行为型模式主要是通过对象关联等方式来分配两个或多个类的职责。根据“合成复用原则”，系统中要尽量使用关联关系来取代继承关系，因此大部分行为型设计模式都属于对象行为型设计模式。

### 命令模式
命令模式(Command Pattern)：将一个请求封装为一个对象，从而使我们可用不同的请求对客户进行参数化；对请求排队或者记录请求日志，以及支持可撤销的操作。命令模式是一种对象行为型模式，其别名为动作(Action)模式或事务(Transaction)模式。

在软件设计中，我们经常需要向某些对象发送请求，但是并不知道请求的接收者是谁，也不知道被请求的操作是哪个，我们只需在程序运行时指定具体的请求接收者即可，此时，可以使用命令模式来进行设计，使得请求发送者与请求接收者消除彼此之间的耦合，让对象之间的调用关系更加灵活。

命令模式可以对发送者和接收者完全解耦，发送者与接收者之间没有直接引用关系，发送请求的对象只需要知道如何发送请求，而不必知道如何完成请求。

所谓对命令的封装，说白了，无非就是把一系列的操作写到一个方法中，然后供客户端调用就行了。

命令模式作为一种行为类模式，首先要做到低耦合，耦合度低了才能提高灵活性，而加入调用者和接收者两个角色的目的也正是为此。

命令模式包含如下角色：
- Command: 抽象命令类，类中对需要执行的命令进行声明，一般来说要对外公布一个execute方法用来执行命令
- ConcreteCommand: 具体命令类
- Invoker: 调用者，负责调用命令。
- Receiver: 接收者，负责接收命令并且执行命令。
- Client:客户类

```php
<?php
//命令
interface Command
{
    public function execute();
}

class ConcreteCommand implements Command
{
    private $receiver;

    public function __construct(Receiver $receiver)
    {
        $this->receiver = $receiver;
    }

    public function execute()
    {
        $this->receiver->doSomething();
    }
}

//调用者
class Invoker
{
    /**
     * @var Command
     */
    private $command;

    public function setCommand(Command $command)
    {
        $this->command = $command;
    }

    public function action()
    {
        $this->command->execute();
    }
}


//接受者
class Receiver
{
    public function doSomething()
    {
        echo '接受者 do something';
    }
}

$receiver = new Receiver();
$command = new ConcreteCommand($receiver);
$command->execute();

$invoker = new Invoker();
$invoker->setCommand($command);
$invoker->action();
```

### 中介者模式
中介者模式(Mediator Pattern)定义：用一个中介对象来封装一系列的对象交互，中介者使各对象不需要显式地相互引用，从而使其耦合松散，而且可以独立地改变它们之间的交互。中介者模式又称为调停者模式，它是一种对象行为型模式。

一般来说，同事类之间的关系是比较复杂的，多个同事类之间互相关联时，他们之间的关系会呈现为复杂的网状结构，这是一种过度耦合的架构，即不利于类的复用，也不稳定。

如果引入中介者模式，那么同事类之间的关系将变为星型结构，任何一个类的变动，只会影响的类本身，以及中介者，这样就减小了系统的耦合。一个好的设计，必定不会把所有的对象关系处理逻辑封装在本类中，而是使用一个专门的类来管理那些不属于自己的行为。

中介者模式包含如下角色：
- Mediator: 抽象中介者
- ConcreteMediator: 具体中介者
- Colleague: 抽象同事类
- ConcreteColleague: 具体同事类

有两个类A和B，类中各有一个数字，并且要保证类B中的数字永远是类A中数字的100倍。也就是说，当修改类A的数时，将这个数字乘以100赋给类B，而修改类B时，要将数除以100赋给类A。类A类B互相影响，就称为同事类。
```php
<?php

abstract class AbstractMediator
{
    protected $colleagueA;
    protected $colleagueB;

    public function __construct(AbstractColleague $a, AbstractColleague $b)
    {
        $this->colleagueA = $a;
        $this->colleagueB = $b;
    }

    abstract public function AaffectB();

    abstract public function BaffectA();
}

class Mediator extends AbstractMediator
{
    public function AaffectB()
    {
        $number = $this->colleagueA->getNumber();
        $this->colleagueB->changeNumber($number * 100);
    }

    public function BaffectA()
    {
        $number = $this->colleagueB->getNumber();
        $this->colleagueA->changeNumber($number / 100);
    }
}

abstract class AbstractColleague
{
    protected $number;

    public function getNumber()
    {
        return $this->number;
    }

    public function changeNumber($number)
    {
        $this->number = $number;
    }

    abstract public function setNumber($number, AbstractMediator $mediator);
}

class ColleagueA extends AbstractColleague
{
    public function setNumber($number, AbstractMediator $mediator)
    {
        $this->number = $number;
        $mediator->AaffectB();
    }
}

class ColleagueB extends AbstractColleague
{
    public function setNumber($number, AbstractMediator $mediator)
    {
        $this->number = $number;
        $mediator->BaffectA();
    }
}

$a = new ColleagueA();
$b = new ColleagueB();
$m = new Mediator($a, $b);
$a->setNumber(35, $m);
echo $b->getNumber();

```

### 观察者模式
观察者模式(Observer Pattern)：定义对象间的一种一对多依赖关系，使得每当一个对象状态发生改变时，其相关依赖对象皆得到通知并被自动更新。观察者模式又叫做发布-订阅（Publish/Subscribe）模式、模型-视图（Model/View）模式、源-监听器（Source/Listener）模式或从属者（Dependents）模式。

建立一种对象与对象之间的依赖关系，一个对象发生改变时将自动通知其他对象，其他对象将相应做出反应。在此，发生改变的对象称为观察目标，而被通知的对象称为观察者，一个观察目标可以对应多个观察者，而且这些观察者之间没有相互联系，可以根据需要增加和删除观察者，使得系统更易于扩展，这就是观察者模式的模式动机。

观察者模式包含如下角色：
- Subject: 目标，抽象主题角色把所有观察者对象保存在一个集合里，每个主题都可以有任意数量的观察者，抽象主题提供一个接口，可以增加和删除观察者对象。
- ConcreteSubject: 具体目标，该角色将有关状态存入具体观察者对象，在具体主题的内部状态发生改变时，给所有注册过的观察者发送通知。
- Observer: 观察者，是观察者者的抽象类，它定义了一个更新接口，使得在得到主题更改通知时更新自己。
- ConcreteObserver: 具体观察者
```php
//观察者 Observer
interface Observer
{
    public function update($message);
}

class ConcreteObserverA implements Observer
{
    public function update($message)
    {
        echo '观察者A收到消息' . $message;
    }
}

class ConcreteObserverB implements Observer
{
    public function update($message)
    {
        echo '观察者B收到消息' . $message;
    }
}

//被观察者 Subject
interface Subject
{
    //增加订阅者
    public function attach(Observer $observer);

    //删除订阅者
    public function detach(Observer $observer);

    //通知订阅者更新消息
    public function notify($message);
}

class SubscriptionSubject implements Subject
{
    //观察者
    private $observers = [];

    public function attach(Observer $observer)
    {
        $this->observers[get_class($observer)] = $observer;
    }

    public function detach(Observer $observer)
    {
        unset($this->observers[get_class($observer)]);
    }

    public function notify($message)
    {
        foreach ($this->observers as $observer) {
            $observer->update($message);
        }
    }
}

$a = new ConcreteObserverA();
$b = new ConcreteObserverB();
$s = new SubscriptionSubject();
$s->attach($a);
$s->attach($b);
$s->notify('你好啊');
```



###状态模式
状态模式(State Pattern) ：允许一个对象在其内部状态改变时改变它的行为，对象看起来似乎修改了它的类。其别名为状态对象(Objects for States)，状态模式是一种对象行为型模式。

在很多情况下，一个对象的行为取决于一个或多个动态变化的属性，这样的属性叫做状态，这样的对象叫做有状态的(stateful)对象，这样的对象状态是从事先定义好的一系列值中取出的。当一个这样的对象与外部事件产生互动时，其内部状态就会改变，从而使得系统的行为也随之发生变化。

状态模式包含如下角色：
- Context: 环境类，又称为上下文类，它是拥有多种状态的对象。由于环境类的状态存在多样性且在不同状态下对象的行为有所不同，因此将状态独立出去形成单独的状态类。在环境类中维护一个抽象状态类State的实例，这个实例定义当前状态，在具体实现时，它是一个State子类的对象。
- State: 抽象状态类，用于定义一个接口以封装与环境类的一个特定状态相关的行为，在抽象状态类中声明了各种不同状态对应的方法，而在其子类中实现类这些方法，由于不同状态下对象的行为可能不同，因此在不同子类中方法的实现可能存在不同，相同的方法可以写在抽象状态类中。
- ConcreteState: 具体状态类，它是抽象状态类的子类，每一个子类实现一个与环境类的一个状态相关的行为，每一个具体状态类对应环境的一个具体状态，不同的具体状态类其行为有所不同。

```php
//State（抽象状态类）
interface State
{
    public function handleState();
}

//ConcreteState（具体状态类）
class ConcreteStateA implements State
{
    public function handleState()
    {
        echo "State: A";
    }
}

class ConcreteStateB implements State
{
    public function handleState()
    {
        echo "State: B";
    }
}

//Context（环境类）
class Context
{
    /**
     * @var State
     */
    private $state;

    public function setState(State $state)
    {
        $this->state = $state;
    }

    public function operation()
    {
        $this->state->handleState();
    }
}

$context = new Context();
$context->setState(new ConcreteStateA());
$context->operation();

$context->setState(new ConcreteStateB());
$context->operation();
```

###策略模式
策略模式(Strategy Pattern)：定义一系列算法，将每一个算法封装起来，并让它们可以相互替换。策略模式让算法独立于使用它的客户而变化，也称为政策模式(Policy)。

完成一项任务，往往可以有多种不同的方式，每一种方式称为一个策略，我们可以根据环境或者条件的不同选择不同的策略来完成该项任务。

在软件开发中也常常遇到类似的情况，实现某一个功能有多个途径，此时可以使用一种设计模式来使得系统可以灵活地选择解决途径，也能够方便地增加新的解决途径。

在软件系统中，有许多算法可以实现某一功能，如查找、排序等，一种常用的方法是硬编码(Hard Coding)在一个类中，如需要提供多种查找算法，可以将这些算法写到一个类中，在该类中提供多个方法，每一个方法对应一个具体的查找算法；当然也可以将这些查找算法封装在一个统一的方法中，通过if…else…等条件判断语句来进行选择。这两种实现方法我们都可以称之为硬编码，如果需要增加一种新的查找算法，需要修改封装算法类的源代码；更换查找算法，也需要修改客户端调用代码。在这个算法类中封装了大量查找算法，该类代码将较复杂，维护较为困难。

除了提供专门的查找算法类之外，还可以在客户端程序中直接包含算法代码，这种做法更不可取，将导致客户端程序庞大而且难以维护，如果存在大量可供选择的算法时问题将变得更加严重。

为了解决这些问题，可以定义一些独立的类来封装不同的算法，每一个类封装一个具体的算法，在这里，每一个封装算法的类我们都可以称之为策略(Strategy)，为了保证这些策略的一致性，一般会用一个抽象的策略类来做算法的定义，而具体每种算法则对应于一个具体策略类。

策略模式包含如下角色：
- Context: 环境类，也叫上下文，对策略进行二次封装，目的是避免高层模块对策略的直接调用。
- Strategy: 抽象策略类
- ConcreteStrategy: 具体策略类，由一组封装了算法的类来担任，这些类之间可以根据需要自由替换
```php
interface Strategy
{
    public function sendEmail($email);
}

class ConcreteStrategyA implements Strategy
{
    public function sendEmail($email)
    {
        echo "使用策略A发邮件，发送给了{$email}";
    }
}

class ConcreteStrategyB implements Strategy
{
    public function sendEmail($email)
    {
        echo "使用策略B发邮件，发送给了{$email}";
    }
}

class Context
{
    private $strategy;
    public function __construct(Strategy $strategy)
    {
        $this->strategy = $strategy;
    }

    public function send($email)
    {
        $this->strategy->sendEmail($email);
    }
}

$context = new Context(new ConcreteStrategyA());
$context->send('aa@qq.com');

$context = new Context(new ConcreteStrategyB());
$context->send('aa@qq.com');
```

参考：
- https://github.com/me115/design_patterns
- https://www.zhihu.com/question/20367734