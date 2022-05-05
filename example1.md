# S.O.L.I.D: The First 5 Principles of OOD
Solid is composition for the first five object-oriented design(OOD) principles by Robert C. Martin, popularly known as Uncle Bob.
What Solid Stands For:
- __S —__ Single responsibility principle
- __O —__ Open-closed principle
- __L —__ Liskov substitution principle
- __I —__ Interface segregation principle
- __D —__ Dependency Inversion Principle

<br>

## Single Responsibility Principle (SRP):
A Class should be responsible for a single task or a class must have a specific purpose.

Lets see an example for this

I have a class order which is responsible for different operations like calculate the sum , get items , print items , show items , save order , delete order etc

<br>

```PHP
class Order
{
    public function calculateTotalSum(){ }
    public function getItems(){ }
    public function getItemCount(){ }
    public function addItem($item){ }
    public function deleteItem($item){ }

    public function printOrder(){ }
    public function showOrder(){ }

    public function load(){ }
    public function save(){ }
    public function update(){ }
    public function delete(){ }
}
```

<br>

For smaller class you might think it’s fine but it can quickly lead to inoperability. 

For example, later on if we had a problem with printing, we have to revisit this class that is doing much more than just printing. The idea behind this principle is that each of your classes, modules, methods are responsible for one and only one thing.

Let’s update our example and see how we can implement this principle in this example of ours.

<br>

```PHP
class Order
{
    public function calculateTotalSum(){ }
    public function getItems(){ }
    public function getItemCount(){ }
    public function addItem($item){ }
    public function deleteItem($item){ }
}

class OrderRepository
{
    public function load($orderID){ }
    public function save($order){ }
    public function update($order){ }
    public function delete($order){ }
}

class OrderViewer
{
    public function printOrder($order){ }
    public function showOrder($order){ }
}

```
<br>

As you can see, now we ended up having three separate classes where each class is responsible for one and only one thing. Later on if we have an issue with the printing, we know where to look at, if we have problem with the CRUD, we know we have to look at the `OrderRepository` and same goes for the business logic.

<br>

## Open-Close Principle:
This principle states that Objects or entities should be open for extension, but closed for modification. This simply means that a class should be easily extendable without modifying the class itself

Now lets take the same example for __OrderRepository__ class

<br>

```PHP
class OrderRepository
{
    public function load($orderID)
    {
        return DB:table('order')->findorFail($orderID);
    }
    
    public function save($order){ }
    public function update($order){ }
    public function delete($order){ }
}
```

<br>

In this case we have data source for orders from database but if we want to load the data from API from a third party api server then what will be steps. Either we need to implement in the same class which will be a mess and it does not also fulfill the Open Closed Principle.

So we can implement in this way we can create an interface __OrderSource__ and then create two different classes `DbOrderSource` and `ApiOrderSource` which will be implementing the OrderSource interface. Then in the OrderRepository class we can set the source where you want to get the data

<br>

```PHP
class OrderRepository
{
    private $source;
    public function setSource(OrderSource $source)
    {
        $this->source = $source;
    }
    public function load($orderID)
    {
        return $this->source->load($orderID);
    }
    public function save($order){ }
    public function update($order){ }
}

interface OrderSource
{
    public function load($orderID);
    public function save($order);
    public function update($order);
    public function delete($order);
}

class DbOrderSource implements OrderSource
{
    public function load($orderID);
    public function save($order){ }
    public function update($order){ }
    public function delete($order){ }
}

class ApiOrderSource implements OrderSource
{
    public function load($orderID);
    public function save($order){ }
    public function update($order){ }
    public function delete($order){ }
}
```

<br>

Now we can change the source of the order by just setting the different source in `OrderRepository` class even in future if we want to introduce any source we will implement the new class and implement the same interface and then we can set the source in OrderRepository class and use this source and it complies with the Open-Closed Principle of SOLID.

<br>

## Liskov substitution principle:
Liskov substitution principle says every class that inherit from a parent class, must not replicate functionality already implemented in the parent class.

Then the parent class should be able to be replaced by any of its subclasses in any region of the code.

Lets say we have our class Rectangle, and we want to create a class Square that extends from Rectangle.

<br>

```PHP
class Rectangle
{
    public function setWidth($w) 
    { 
        $this->width = $w; 
    }
    public function setHeight($h) 
    { 
        $this->height = $h; 
    }
    public function getArea() 
    { 
       return $this->height * $this->width; 
    }
}
class Square extends Rectangle
{
    public function setWidth($w) 
    {
        $this->width = $w; 
        $this->height = $w; 
    }
    public function setHeight($h) 
    { 
        $this->height = $h; 
        $this->width = $h; 
    }
}
```

And we create a function to calculate the area of rectangle

```PHP
function areaOfRectangle() 
{
    $rectangle = new Rectangle();
    $r->setWidth(7); $r->setHeight(3);
    $r->getArea(); // 21
}

```

as the Liskov substitution principle says, we should be able to change Rectangle by Square class so lets try it

```PHP
function areaOfRectangle() 
{
     $rectangle = new Square();
     $r->setWidth(7); 
     $r->setHeight(3);
     $r->getArea(); // 9
}
```
<br>

Now just Recall what we have discussed earlier “we need to make sure that Square classes are extending the Rectangle without changing their behaviour”. But, now we get different results, 21 is not equal to 9.

So we can solve this issue by introducing the an interface.

<br>

```PHP
interface Polygon
{
    public function setHeight($h);
    public function setWidht($w);
    public function getArea();
}
class Rectangle implements Polygon { };
 
class Square implements Polygon { };
```

In Liskov Subtitution Principle you need to follow the correct hierarchy for your classes if you do not follow it the unit test for the superclass would never success for the subclasses.

<br>

## Interface segregation principle:
Interface segregation principle states that many specialized interfaces are better than one universal. In other words we can say this also that client must not be forced to implement an interface that it doesn’t use.

So the main purpose is to divide the interfaces so that they are more specific.

Lets discuss an example for an a store lets say we have products where we can have the promotions in terms of codes and discounts and they have same price conditions and also colors and size etc so the Product interface will be like this

<br>

```PHP
interface Product
{
    public function applyDiscount($discount);
    public function applyPromocode($promocode);
    public function setColor($color);
    public function setSize($size);
    
    public function setCondition($condition);
    public function setPrice($price);
}
```
<br>

This interface has so many issues because it injects so many methods which is not necessary under certain conditions like what is our product does not have any promotion code applicable or discount

So in order not to implement the methods in the interface which is not used its good to break the interface and make multiple interfaces into several smaller interfaces.

<br>

```PHP
interface Product
{
    public function setCondition($condition);
    public function setPrice($price);
}
interface Clothes
{
    public function setColor($color);
    public function setSize($size);
    public function setMaterial($material);
}
interface Discountable
{
    public function applyDiscount($discount);
    public function applyPromocode($promocode);
}
class Book implemets Product, Discountable
{
    public function setCondition($condition){ }
    public function setPrice($price){ }
    public function applyDiscount($discount){ }
    public function applyPromocode($promocode){ }
}
class MenClothes implemets Product, Clothes
{
    public function setCondition($condition){ }
    public function setPrice($price){ }
    public function setColor($color){ }
    public function setSize($size){ }
    public function setMaterial($material){ }
}
```

Hence conclusion is class should not have the extra knowledge that is not required.

<br>

## Dependency Inversion Principle:
Dependency inversion principle states that
High level modules should not depend on low-level modules, both should depend on abstractions.

Abstractions should not depend on details. Details should depend on abstractions.

Or it can be rephrases as “the dependencies should be based on abstractions, not details.”

Lets see and example We have the Manager class, which is the high level class, and then a class called Worker. The Manager can make Workers to work. Let’s see a traditional OO implementation:

<br>

```PHP
class worker
{
  public function work(){ }
}
class Manager{
 
 private $worker;
 public function setWorker($worker)
 {
    $this->worker = $worker;
 }
 public function manager()
 {
    $this->worker->work();
 }
}
```

<br>

Now, if we need to add a new kind of specialized workers, we create a new class SpecializedWorker for this

<br>

```PHP
class SpecializedWorker
{
 public function work(){ }
}
```

<br>

Now we have few Problems with this implementation

We have to modify the Manager class, and some of the functionality of the Manager might be affected and class will look so fat.

The unit tests should be revise.

We can solve this issue like below example

<br>

```PHP
interface Employee{
    public function work(){}
}
class Worker implements Employee
{
   public function work(){} 
}
class SpecializedWorker implements Employee
{
   public function work(){} 
}
```

<br>

__S.O.L.I.D__ seem to be a handful at first, but with continuous usage it becomes a part of you and your code which can easily be extended, modified, tested, and refactored without any problems.