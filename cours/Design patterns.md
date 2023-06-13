Plan du cours :

**1. Introduction aux Design Patterns**
   - Qu'est-ce qu'un design pattern ?
   - Pourquoi les design patterns sont-ils importants ?
   - Les principes fondamentaux des design patterns.

**2. Catégorisation des Design Patterns**
   - Création de patterns
   - Patterns structurels
   - Patterns comportementaux

**3. Design Patterns de Création**
   - Singleton
   - Builder
   - Prototype
   - Factory Method
   - Abstract Factory

**4. Design Patterns Structurels**
   - Adapter
   - Bridge
   - Composite
   - Decorator
   - Facade
   - Flyweight
   - Proxy

**5. Design Patterns Comportementaux**
   - Chain of Responsibility
   - Command
   - Interpreter
   - Iterator
   - Mediator
   - Memento
   - Observer
   - State
   - Strategy
   - Template Method
   - Visitor

**6. Etude de cas**
   - Application des design patterns dans des projets de programmation réels.
   - Analyse de codes existants : identification et amélioration avec des design patterns.

**7. Anti-patterns**
   - Qu'est-ce qu'un anti-pattern ?
   - Anti-patterns communs et comment les éviter.

**8. Projet Final**
   - Conception et développement d'un projet en utilisant plusieurs design patterns.

# 1. Introduction aux Design Patterns

## 1.1 Qu'est-ce qu'un design pattern?

Un design pattern est une solution éprouvée à un problème courant de conception de logiciels. Il n'est pas un morceau de code réutilisable, mais plutôt un guide pour résoudre certains problèmes de manière générale. Les patterns peuvent accélérer le processus de développement en fournissant des paradigmes de test, des modèles de développement éprouvés.

```mermaid
graph LR
    A[Problèmes de Conception] -->|Résolution| B[Design Patterns]
    B --> C{Catégories de Design Patterns}
    C --> D[Création]
    C --> E[Structurel]
    C --> F[Comportemental]
```

## 1.2 Pourquoi les design patterns sont-ils importants?

Les design patterns sont importants parce qu'ils fournissent des solutions testées et éprouvées à des problèmes de conception fréquemment rencontrés, ils améliorent la lisibilité du code et peuvent rendre le code plus facile à maintenir ou à adapter.

```mermaid
graph TD
    A[Design Patterns] --> B[Solutions Éprouvées]
    A --> C[Lisibilité du Code]
    A --> D[Maintenance du Code]
```

## 1.3 Les principes fondamentaux des design patterns

Il y a quelques principes fondamentaux que tous les design patterns suivent. Ces principes sont :
   
   - **L'Encapsulation** : Les détails spécifiques sont cachés derrière une interface.
   - **La Composition** : Les objets sont composés pour obtenir de nouvelles fonctionnalités.
   - **Le Polymorphisme** : Les entités de base peuvent être remplacées par des entités dérivées.

```mermaid
graph LR
    A[Principes Fondamentaux] --> B{Principes}
    B --> D[Encapsulation]
    B --> E[Composition]
    B --> F[Polymorphisme]
```

Dans les prochaines sections, nous explorerons les divers types de design patterns et nous plongerons dans des exemples spécifiques de chacun.

# 2. Catégorisation des Design Patterns

Les design patterns sont généralement catégorisés en trois types : Création de patterns (ou Creation), Patterns Structurels (ou Structural) et Patterns Comportementaux (ou Behavioral).

```mermaid
graph LR
    A{Design Patterns} --> B[Création de Patterns]
    A --> C[Patterns Structurels]
    A --> D[Patterns Comportementaux]
```

## **2.1 Création de patterns (Creational Patterns)**

Les patterns de création sont conçus pour traiter les problèmes de création d'objets. Ils résolvent ce problème en contrôlant de manière appropriée le processus de création d'objets. Les principaux patterns de cette catégorie incluent : Singleton, Builder, Prototype, Factory Method, et Abstract Factory.

```mermaid
graph TD
    A[Création de Patterns] --> B[Singleton]
    A --> C[Builder]
    A --> D[Prototype]
    A --> E[Factory Method]
    A --> F[Abstract Factory]
```

## **2.2 Patterns Structurels (Structural Patterns)**

Les patterns structurels se préoccupent de la composition des classes et des objets pour former des structures plus grandes. Ils aident à assurer que lorsque les parties d'un système changent, l'ensemble du système n'a pas besoin de changer. Les principaux patterns de cette catégorie incluent : Adapter, Bridge, Composite, Decorator, Facade, Flyweight, et Proxy.

```mermaid
graph TD
    A[Patterns Structurels] --> B[Adapter]
    A --> C[Bridge]
    A --> D[Composite]
    A --> E[Decorator]
    A --> F[Facade]
    A --> G[Flyweight]
    A --> H[Proxy]
```

## **2.3 Patterns Comportementaux (Behavioral Patterns)**

Les patterns comportementaux sont concernés par la communication entre les objets, comment ils interagissent et se répartissent les responsabilités. Les principaux patterns de cette catégorie incluent : Chain of Responsibility, Command, Interpreter, Iterator, Mediator, Memento, Observer, State, Strategy, Template Method, et Visitor.

```mermaid
graph TD
    A[Patterns Comportementaux] --> B[Chain of Responsibility]
    A --> C[Command]
    A --> D[Interpreter]
    A --> E[Iterator]
    A --> F[Mediator]
    A --> G[Memento]
    A --> H[Observer]
    A --> I[State]
    A --> J[Strategy]
    A --> K[Template Method]
    A --> L[Visitor]
```

Dans les sections suivantes, nous aborderons plus en détail chaque catégorie et chaque pattern spécifique.

# 3. Design Patterns de Création

Les Design Patterns de Création se focalisent sur le contrôle de la création des instances. Ces modèles encapsulent la connaissance sur quels objets spécifiques du système doivent être créés, comment ces objets sont créés, et comment ils sont associés.

```mermaid
graph LR
    A{Design Patterns de Création} --> B[Singleton]
    A --> C[Builder]
    A --> D[Prototype]
    A --> E[Factory Method]
    A --> F[Abstract Factory]
```

## 3.1 Singleton

Le Singleton est un pattern de conception qui limite l'instanciation d'une classe à un seul objet. Il est utilisé lorsque vous voulez vous assurer qu'une classe n'a qu'une seule instance, et que vous voulez fournir un point d'accès global à cette instance.

```mermaid
classDiagram
    class Singleton {
        -static instance : Singleton
        -Singleton() : constructor
        +static getInstance() : Singleton
    }
    Singleton --> Singleton : Uses
```

Exemple en python :

Le Singleton peut être implémenté de plusieurs façons. L'une des façons les plus simples est d'utiliser un module. En voici un exemple :

```python
class Singleton:
    _instance = None

    @staticmethod
    def getInstance():
        if Singleton._instance == None:
            Singleton()
        return Singleton._instance

    def __init__(self):
        if Singleton._instance != None:
            raise Exception("Cette classe est un singleton !")
        else:
            Singleton._instance = self

s = Singleton.getInstance()
print(s)

```


## 3.2 Builder

Le pattern Builder ou Constructeur en français est utilisé pour construire des objets complexes étape par étape. Il sépare le code de construction d'un objet de la représentation de cet objet, de sorte que le même code de construction peut créer différents types d'objets.

```mermaid
classDiagram
    class Director {
        +construct() : void
    }
    class Builder {
        +buildPartA() : void
        +buildPartB() : void
        +getResult() : Product
    }
    Director --> Builder : Uses
    Builder <-- ConcreteBuilder : Inherits
    class ConcreteBuilder {
        +buildPartA() : void
        +buildPartB() : void
        +getResult() : Product
    }
```

Le pattern Builder peut être implémenté comme suit :

```python
class Director:
    __builder = None

    def setBuilder(self, builder):
        self.__builder = builder

    def getCar(self):
        car = Car()

        # Build parts
        body = self.__builder.getBody()
        car.setBody(body)

        engine = self.__builder.getEngine()
        car.setEngine(engine)

        return car

# The whole product
class Car:
    def __init__(self):
        self.__engine = None
        self.__body = None

    def setBody(self, body):
        self.__body = body

    def setEngine(self, engine):
        self.__engine = engine

# Abstract Builder
class Builder:
    def getEngine(self): pass
    def getBody(self): pass

# Concrete Builder 
class CarBuilder(Builder):
    def getEngine(self):
        return "Sport engine"

    def getBody(self):
        return "Sport body"

director = Director()
director.setBuilder(CarBuilder())

car = director.getCar()
print(car)
```

## 3.3 Prototype

Le pattern Prototype est utilisé lorsque la création d'un nouvel objet est coûteuse. Il offre un mécanisme pour copier un objet original ou prototype, et ainsi éviter une création coûteuse.

```mermaid
classDiagram
    class Prototype {
        +clone() : Prototype
    }
    Prototype <-- ConcretePrototype : Inherits
```

Le pattern Prototype peut être implémenté grâce à la fonction de copie :

```python
import copy

class Prototype:

    def __init__(self):
        self._objects = {}

    def registerObject(self, name, obj):
        self._objects[name] = obj

    def unregisterObject(self, name):
        del self._objects[name]

    def clone(self, name, **attr):
        obj = copy.deepcopy(self._objects.get(name))
        obj.__dict__.update(attr)
        return obj

class Car:
    def __init__(self):
        self.name = "Car"
        self.color = "Red"
        self.options = "Ex"

car = Car()
prototype = Prototype()
prototype.registerObject("car1", car)

car2 = prototype.clone("car1")
print(car2)
```

## 3.4 Factory Method

Le Factory Method est un pattern qui définit une interface pour créer un objet, mais laisse les sous-classes décider quelles classes instancier. Il permet à une classe de déléguer l'instanciation à des sous-classes.

```mermaid
classDiagram
    class Creator {
        +abstract factoryMethod() : Product
    }
    class Product {
    }
    Creator <-- ConcreteCreator : Inherits
    Product <-- ConcreteProduct : Inherits
    ConcreteCreator --> ConcreteProduct : Creates
```

Le pattern Factory Method peut être implémenté de la manière suivante :

```python
class Dog:
    def __init__(self, name):
        self._name = name

    def speak(self):
        return "Woof!"

class Cat:
    def __init__(self, name):
        self._name = name

    def speak(self):
        return "Meow!"

def get_pet(pet="dog"):
    pets = dict(dog=Dog("Hope"), cat=Cat("Peace"))
    return pets[pet]

d = get_pet("dog")
print(d.speak())

c = get_pet("cat")
print(c.speak())
```

## 3.5 Abstract Factory

L'Abstract Factory est un pattern qui fournit une interface pour créer des familles d'objets liés ou dépendants sans spécifier leurs classes concrètes.

```mermaid
classDiagram
    class AbstractFactory {
        +abstract createProductA() : AbstractProductA
        +abstract createProductB() : AbstractProductB
    }
    class AbstractProductA {
    }
    class AbstractProductB {
    }
    AbstractFactory <-- ConcreteFactory : Inherits
    AbstractProductA <-- ConcreteProductA1 : Inherits
    AbstractProductA <-- ConcreteProductA2 : Inherits
    AbstractProductB <-- ConcreteProductB1 : Inherits
    AbstractProductB <-- ConcreteProductB2 : Inherits
```

Le pattern Abstract Factory peut être implémenté  comme suit :

```python
class Dog:
    def __init__(self, name):
        self._name = name

    def speak(self):
        return "Woof!"

class Cat:
    def __init__(self, name):
        self._name = name

    def speak(self):
        return "Meow!"

class DogFactory:
    def get_pet(self):
        return Dog("Hope")

    def get_food(self):
        return "Dog Food"

class CatFactory:
    def get_pet(self):
        return Cat("Peace")

    def get_food(self):
        return "Cat Food"

class PetStore:
    def __init__(self, pet_factory=None):
        self._pet_factory = pet_factory

    def show_pet(self):
        pet = self._pet_factory.get_pet()
        pet_food = self._pet_factory.get_food()

        print("Our pet is '{}'!".format(pet))
        print("Our pet says hello by '{}'".format(pet.speak()))
        print("Its food is '{}'!".format(pet_food))

#Create a Concrete Factory
factory = DogFactory()

#Create a pet store housing our Abstract Factory
shop = PetStore(factory)

#Invoke the utility method to show the details of our pet
shop.show_pet()
```
Dans cet exemple, le `DogFactory` et le `CatFactory` sont les fabriques concrètes qui produisent des produits cohérents (le `Dog` et la `Dog Food` pour `DogFactory`, le `Cat` et la `Cat Food` pour `CatFactory`). Le `PetStore` est une application qui utilise une Abstract Factory pour créer ses produits.

# 4. Design Patterns Structurels

Les Design Patterns Structurels se focalisent sur la façon dont les classes et les objets sont composés pour former de plus grandes structures. 

```mermaid
graph LR
    A{Design Patterns Structurels} --> B[Adapter]
    A --> C[Bridge]
    A --> D[Composite]
    A --> E[Decorator]
    A --> F[Facade]
    A --> G[Flyweight]
    A --> H[Proxy]
```

## 4.1 Adapter

L'Adapter est un pattern structurel qui permet à des interfaces incompatibles de collaborer. Il convertit l'interface d'une classe en une autre interface que les clients attendent.

```mermaid
classDiagram
    class Target {
        +request() : void
    }
    Target <-- Adapter
    class Adaptee {
        +specificRequest() : void
    }
    Adapter --> Adaptee : Uses
```

Exemple en Python :

```python
class EuropeanSocketInterface:
    def voltage(self): pass

class Socket(EuropeanSocketInterface):
    def voltage(self):
        return 230

class AmericanSocketInterface:
    def voltage(self): pass

class Adapter(EuropeanSocketInterface):
    def __init__(self, socket):
        self.socket = socket

    def voltage(self):
        return 110

american_socket = Adapter(Socket())
print(american_socket.voltage())
```

## 4.2 Bridge

Le Bridge est un pattern qui permet de séparer l'abstraction d'une implémentation, de telle sorte que les deux peuvent être modifiées indépendamment.

```mermaid
classDiagram
    class Abstraction {
        +operation() : void
    }
    class Implementation {
        +operationImpl() : void
    }
    Abstraction o--> Implementation : Uses
    class RefinedAbstraction {
        +operation() : void
    }
    Abstraction <|-- RefinedAbstraction
    class ConcreteImplementationA {
        +operationImpl() : void
    }
    Implementation <|-- ConcreteImplementationA
    class ConcreteImplementationB {
        +operationImpl() : void
    }
    Implementation <|-- ConcreteImplementationB
```

Exemple en Python :

```python
class ImplementationInterface:
    def operation_implementation(self):
        pass

class Abstraction:
    def __init__(self, implementation):
        self.implementation = implementation

    def operation(self):
        self.implementation.operation_implementation()

class ImplementationA(ImplementationInterface):
    def operation_implementation(self):
        return 'Implementation A'

class ImplementationB(ImplementationInterface):
    def operation_implementation(self):
        return 'Implementation B'

a = Abstraction(ImplementationA())
b = Abstraction(ImplementationB())
print(a.operation())
print(b.operation())
```

## 4.3 Composite

Le Composite est un pattern de conception qui permet de traiter les objets individuels et les compositions d'objets de manière uniforme.

```mermaid
classDiagram
    class Component {
        +operation() : void
    }
    class Leaf {
        +operation() : void
    }
    Component <|-- Leaf
    class Composite {
        +add(component : Component) : void
        +remove(component : Component) : void
        +operation() : void
    }
    Component <|-- Composite
    Component <.. Composite : Contains
```

Exemple en Python :
Pardon pour l'interruption précédente. Je vais reprendre l'exemple du Composite en Python.

```python
class Component:
    def __init__(self, name):
        self._name = name

    def component_function(self):
        print(f"{self._name}")

class Child(Component):
    def __init__(self, name):
        super().__init__(name)

    def component_function(self):
        super().component_function()

class Composite(Component):
    def __init__(self, name):
        super().__init__(name)
        self._children = []

    def add(self, child):
        self._children.append(child)

    def remove(self, child):
        self._children.remove(child)

    def component_function(self):
        print(f"{self._name}")
        for i in self._children:
            i.component_function()

sub1 = Composite("submenu1")
sub11 = Child("sub_submenu 11")
sub12 = Child("sub_submenu 12")

sub1.add(sub11)
sub1.add(sub12)

top = Composite("top_menu")
sub2 = Child("submenu2")

top.add(sub1)
top.add(sub2)
top.component_function()
```

* **4.4 Decorator**

Le Decorator est un pattern de conception qui permet d'ajouter de nouveaux comportements à des objets en les plaçant dans des objets enveloppeurs spéciaux.

```mermaid
classDiagram
    class Component {
        +operation() : void
    }
    class ConcreteComponent {
        +operation() : void
    }
    Component <|-- ConcreteComponent
    class Decorator {
        -component : Component
        +Decorator(c : Component) : void
        +operation() : void
    }
    Component <|-- Decorator
    Decorator o--> Component
    class ConcreteDecoratorA {
        -addedState : string
        +operation() : void
    }
    Decorator <|-- ConcreteDecoratorA
```

Exemple en Python :

```python
class Component:
    def operation(self):
        pass

class ConcreteComponent(Component):
    def operation(self):
        return "ConcreteComponent"

class Decorator(Component):
    _component = None

    def __init__(self, component):
        self._component = component

    def operation(self):
        self._component.operation()

class ConcreteDecoratorA(Decorator):
    def operation(self):
        return f"ConcreteDecoratorA({self._component.operation()})"

simple = ConcreteComponent()
decorator = ConcreteDecoratorA(simple)
print(decorator.operation())
```

## 4.5 Facade

La Facade est un pattern de conception structurel qui offre une interface simplifiée à une bibliothèque, un framework ou tout autre ensemble complexe de classes.

```mermaid
classDiagram
    class Facade {
        +operation() : void
    }
    class ComplexSubsystemClass1 {
        +operation1() : void
    }
    class ComplexSubsystemClass2 {
        +operation2() : void
    }
    Facade --> ComplexSubsystemClass1 : Uses
    Facade --> ComplexSubsystemClass2 : Uses
```

Exemple en Python :

```python
class ComplexSubsystem1:
    def operation1(self):
        return "Subsystem1: Ready!"

class ComplexSubsystem2:
    def operation1(self):
        return "Subsystem2: Ready!"

class Facade:
    def __init__(self, subsystem1, subsystem2):
        self._subsystem1 = subsystem1 or ComplexSubsystem1()
        self._subsystem2 = subsystem2 or ComplexSubsystem2()

    def operation(self):
        results = []
        results.append(self._subsystem1.operation1())
        results.append(self._subsystem2.operation1())
        return "\n".join(results)

subsystem1 = ComplexSubsystem1()
subsystem2 = ComplexSubsystem2()
facade = Facade(subsystem1, subsystem2)
print(facade.operation())
```

## 4.6 Flyweight

Le Flyweight est un pattern de conception structurel qui permet d'adapter plus d'objets dans la quantité de RAM disponible en partageant des parties communes des états d'objets entre plusieurs objets.

```mermaid
classDiagram
    class Flyweight {
        -intrinsicState : string
        +operation(extrinsicState) : void
    }
    class UnsharedConcreteFlyweight {
        -allState : string
        +operation(extrinsicState) : void
    }
    Flyweight <|-- UnsharedConcreteFlyweight
    class FlyweightFactory {
        -flyweights : dict
        +getFlyweight(key) : Flyweight
    }
    FlyweightFactory o--> Flyweight
```

Exemple en Python :

```python
class Flyweight:
    def operation(self, extrinsic_state):
        pass

class ConcreteFlyweight(Flyweight):
    def operation(self, extrinsic_state):
        return f"ConcreteFlyweight: {str(extrinsic_state)}"

class FlyweightFactory:
    _flyweights = {}

    def get_flyweight(self, key):
        try:
            flyweight = self._flyweights[key]
        except KeyError:
            flyweight = ConcreteFlyweight()
            self._flyweights[key] = flyweight
        return flyweight

factory = FlyweightFactory()
flyweight = factory.get_flyweight("key")
print(flyweight.operation([1, 2, 3]))
```

## 4.7 Proxy

Le Proxy est un pattern de conception structurel qui offre un substitut ou un espace réservé pour un autre objet. Un Proxy contrôle l'accès à l'objet original, permettant d'effectuer quelque chose soit avant soit après la demande atteint l'objet original.

```mermaid
classDiagram
    class Subject {
        +request() : void
    }
    class RealSubject {
        +request() : void
    }
    Subject <|-- RealSubject
    class Proxy {
        -realSubject : RealSubject
        +request() : void
    }
    Subject <|-- Proxy
    Proxy o--> RealSubject
```

Exemple en Python :

Excusez-moi pour l'interruption. Je vais reprendre avec l'exemple de Proxy en Python :

```python
class RealSubject:
    def request(self):
        return "RealSubject: Handling request."

class Proxy:
    def __init__(self, real_subject):
        self._real_subject = real_subject

    def request(self):
        if self.check_access():
            self._real_subject.request()
            self.log_access()

    def check_access(self):
        print("Proxy: Checking access prior to firing a real request.")
        return True

    def log_access(self):
        print("Proxy: Logging the time of request.", end="")

real_subject = RealSubject()
proxy = Proxy(real_subject)
proxy.request()
```
Dans cet exemple, le `Proxy` est utilisé pour contrôler l'accès à `RealSubject`. Avant de transmettre la demande à `RealSubject`, `Proxy` effectue d'autres actions comme la vérification des droits d'accès et le journalisation.

# 5. Design Patterns Comportementaux

Ces modèles sont centrés sur la communication et la délégation de responsabilités entre les objets.

```mermaid
graph LR
    A{Design Patterns Comportementaux} --> B[Chain of Responsibility]
    A --> C[Command]
    A --> D[Interpreter]
    A --> E[Iterator]
    A --> F[Mediator]
    A --> G[Memento]
    A --> H[Observer]
    A --> I[State]
    A --> J[Strategy]
    A --> K[Template Method]
    A --> L[Visitor]
```

Commençons par le premier de notre liste.

* **5.1 Chain of Responsibility**

Ce modèle crée une chaîne d'objets récepteurs pour une requête. Cette chaîne de responsabilité passe la requête le long de la chaîne jusqu'à ce qu'un objet la traite.

```mermaid
classDiagram
    class Handler {
        +set_next(handler : Handler) : Handler
        +handle(request) : string
    }
    Handler <|-- ConcreteHandler1
    Handler <|-- ConcreteHandler2
    class Client {
        -handler : Handler
        +Client() : void
        +do_something() : void
    }
    Client o--> Handler : Uses >
```

Voici un exemple de ce modèle en Python :

```python
class Handler:
    _next_handler = None

    def set_next(self, handler):
        self._next_handler = handler
        return handler

    def handle(self, request):
        if self._next_handler:
            return self._next_handler.handle(request)
        return None

class ConcreteHandler1(Handler):
    def handle(self, request):
        if request == "request1":
            return "ConcreteHandler1"
        else:
            return super().handle(request)

class ConcreteHandler2(Handler):
    def handle(self, request):
        if request == "request2":
            return "ConcreteHandler2"
        else:
            return super().handle(request)

handler1 = ConcreteHandler1()
handler2 = ConcreteHandler2()
handler1.set_next(handler2)

print(handler1.handle("request2"))
```

Dans cet exemple, nous avons deux gestionnaires : `ConcreteHandler1` et `ConcreteHandler2`. Chaque gestionnaire vérifie si elle peut traiter la requête. Si elle ne le peut pas, elle passe la requête au prochain gestionnaire de la chaîne.

## 5.2 Command

Le modèle de Commande transforme une requête en un objet autonome qui contient toute l'information nécessaire à la requête. Cette transformation permet de paramétrer des méthodes avec des files d'attente, des demandes et des opérations, de retarder l'exécution d'une commande et de soutenir les opérations réversibles.

```mermaid
classDiagram
    class Invoker {
        -command : Command
        +set_command(c : Command) : void
        +do_something() : void
    }
    class Command {
        +execute() : void
    }
    Command <|-- ConcreteCommand
    class Receiver {
        +do_something() : void
    }
    ConcreteCommand --> Receiver
    Invoker o--> Command
```

Voici un exemple de ce modèle en Python :

```python
class Command:
    def execute(self):
        pass

class ConcreteCommand(Command):
    def __init__(self, receiver):
        self._receiver = receiver

    def execute(self):
        self._receiver.do_something()

class Receiver:
    def do_something(self):
        print("Receiver is doing something.")

class Invoker:
    def set_command(self, command):
        self._command = command

    def do_something(self):
        self._command.execute()

receiver = Receiver()
command = ConcreteCommand(receiver)
invoker = Invoker()
invoker.set_command(command)
invoker.do_something()
```

## 5.3 Interpreter

L'Interpréteur est un modèle de conception comportemental qui spécifie comment évaluer des phrases dans une langue. Ce modèle implique la mise en œuvre d'un processeur de langage de programmation, peut être utilisé pour développer un compilateur ou un interpréteur.

```mermaid
classDiagram
    class AbstractExpression {
        +interpret(context : Context) : void
    }
    class TerminalExpression {
        +interpret(context : Context) : void
    }
    class NonterminalExpression {
        +interpret(context : Context) : void
    }
    AbstractExpression <|-- TerminalExpression
    AbstractExpression <|-- NonterminalExpression
    class Context {
        -input : string
        -output : string
    }
    NonterminalExpression o--> Context
    TerminalExpression o--> Context
```

Exemple en Python :

```python
class Context:
    def __init__(self, input):
        self.input = input
        self.output = 0

class AbstractExpression:
    def interpret(self, context):
        pass

class NonterminalExpression(AbstractExpression):
    def interpret(self, context):
        context.output = int(context.input) * 2

class TerminalExpression(AbstractExpression):
    def interpret(self, context):
        context.output = int(context.input) + 1

context = Context("5")
list_of_expressions = []
list_of_expressions.append(NonterminalExpression())
list_of_expressions.append(TerminalExpression())

for expression in list_of_expressions:
    expression.interpret(context)

print(context.output)
```
Dans cet exemple, `NonterminalExpression` double la valeur de `input` et `TerminalExpression` l'incrémente.

## 5.4 Iterator

L'itérateur est un modèle de conception comportemental qui permet de parcourir les éléments d'un objet complexe sans exposer sa représentation sous-jacente.

```mermaid
classDiagram
    class Iterator {
        +next() : Element
        +has_next() : boolean
    }
    class ConcreteIterator {
        -collection : Collection
        -position : int
        +next() : Element
        +has_next() : boolean
    }
    Iterator <|-- ConcreteIterator
    class Aggregate {
        +create_iterator() : Iterator
    }
    class ConcreteAggregate {
        -elements : list
        +create_iterator() : Iterator
    }
    Aggregate <|-- ConcreteAggregate
    Aggregate o--> Iterator : Creates >
    ConcreteIterator --> ConcreteAggregate : Uses >
```

Exemple en Python :
Attention ! Ce code est donné à titre d'exemple, car le patron de modèle "Iterator" fait parti de la définition du langage python, en conséquence il est préférable d'utiliser ceux fournis par les bibliothèques standards.
```python
class Iterator:
    def next(self):
        pass

    def has_next(self):
        pass

class ConcreteIterator(Iterator):
    def __init__(self, collection):
        self._collection = collection
        self._position = 0

    def next(self):
        try:
            result = self._collection[self._position]
            self._position += 1
        except IndexError:
            result = None
        return result

    def has_next(self):
        return self._position < len(self._collection)

class Aggregate:
    def create_iterator(self):
        pass

class ConcreteAggregate(Aggregate):
    def __init__(self):
        self._elements = ["Element 1", "Element 2", "Element 3"]

    def create_iterator(self):
        return ConcreteIterator(self._elements)

aggregate = ConcreteAggregate()
iterator = aggregate.create_iterator()

while iterator.has_next():
    print(iterator.next())
```

Dans cet exemple, `ConcreteIterator` itère sur `ConcreteAggregate`, qui est une collection d'éléments de chaînes.

## 5.5 Mediator

Le modèle Mediator définit un objet qui encapsule comment un ensemble d'objets interagit. Il est utile pour promouvoir un couplage faible en évitant que les objets communiquent explicitement entre eux.

```mermaid
classDiagram
    class Mediator {
        +notify(sender : Component, event : string) : void
    }
    class ConcreteMediator {
        -component1 : Component1
        -component2 : Component2
        +notify(sender : Component, event : string) : void
    }
    Mediator <|-- ConcreteMediator
    class Component {
        -mediator : Mediator
        +set_mediator(mediator : Mediator) : void
    }
    class Component1 {
        +do_something() : void
        +do_something_else() : void
    }
    class Component2 {
        +do_something() : void
        +do_something_else() : void
    }
    Component <|-- Component1
    Component <|-- Component2
    Component1 --> Mediator
    Component2 --> Mediator
    ConcreteMediator --> Component1 : Knows >
    ConcreteMediator --> Component2 : Knows >
```

Exemple en Python :

```python
class Mediator:
    def notify(self, sender, event):
        pass

class ConcreteMediator(Mediator):
    def notify(self, sender, event):
        if sender == "component1" and event == "event1":
            print("Mediator reacts on event1 and triggers following operations:")
        elif sender == "component2" and event == "event2":
            print("Mediator reacts on event2 and triggers following operations:")

class Component:
    def __init__(self, mediator):
        self._mediator = mediator

class Component1(Component):
    def do_something(self):
        self._mediator.notify("component1", "event1")

class Component2(Component):
    def do_something(self):
        self._mediator.notify("component2", "event2")

mediator = ConcreteMediator()
component1 = Component1(mediator)
component2 = Component2(mediator)
component1.do_something()
component2.do_something()
```
Dans cet exemple, `ConcreteMediator` coordonne les actions entre `Component1` et `Component2` en réponse à des notifications.

## 5.6 Memento

Le modèle Memento est utilisé pour restaurer l'état d'un objet à un moment précédent.

```mermaid
classDiagram
    class Originator {
        -state : string
        +do_something() : void
        +save() : Memento
        +restore(memento : Memento) : void
    }
    class Memento {
        -state : string
        +get_state() : string
    }
    class Caretaker {
        -mementos : list
        -originator : Originator
        +backup() : void
        +undo() : void
    }
    Caretaker o--> Originator
    Originator --> Memento
    Caretaker --> Memento
```

Exemple en Python :

```python
import datetime

class Memento:
    def __init__(self, state):
        self._state = state
        self._date = str(datetime.datetime.now())

    def get_state(self):
        return self._state

class Originator:
    _state = ""

    def do_something(self):
        self._state = "new state"

    def save(self):
        return Memento(self._state)

    def restore(self, memento):
        self._state = memento.get_state()

class Caretaker:
    def __init__(self, originator):
        self._mementos = []
        self._originator = originator

    def backup(self):
        self._mementos.append(self._originator.save())

    def undo(self):
        if not len(self._mementos):
            return
        memento = self._mementos.pop()
        self._originator.restore(memento)

originator = Originator()
caretaker = Caretaker(originator)

originator.do_something()
caretaker.backup()

originator.do_something()
caretaker.backup()

caretaker.undo()
```

Dans cet exemple, chaque fois que `originator` fait quelque chose, `caretaker` fait une sauvegarde. `Caretaker` peut annuler les actions de `originator` en restaurant son état à partir d'un memento.

@TODO continuer