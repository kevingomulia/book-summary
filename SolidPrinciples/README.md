### SOLID Principles

#### Single Responsibility Principle (SRP)
**>> A class should have one, and only one, reason to change.**

The Single Responsibility Principle (SRP) allows developers to build robust and maintainable software.  
It is applicable not only to classes, but also software components and microservices.

**Benefits**  
##### Reduces the effect changes make on your code
Classes that implement more responsibilities are more difficult to maintain cleanly than classes that only do one thing.  
SRP reduces the number of dependent classes affected by your code change.

##### Easier to understand
Reduces number of bugs and improves development speed.

##### Ask a question to validate your design
Ask yourself: "what is the responsibility of this class/microservice?"  
If your answer includes the word "and", you're most likely breaking the SRP.  
Take a step back and rethink your current design. There is probably a better way to implement it.

**Examples of SRP**

**Java Persistence API (JPA) specification** has only one responsibility:   
Defining a standardized way to manage data persisted in a relational database by using the object-relational mapping concept.

It is a huge responsibility, but it is the ONLY responsibility.

**JPA EntityManager** interface provides a set of methods to persist, update, remove and read entities from a RDBMS. Its responsibility is to manage the entities that are associated with the current persistence context.

#### Open/Closed Principle (OCP)
**>> Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification.**

General idea: write your code so that you will be able to add new functionality without changing the existing code.  

Initially, Bertrand Mayer proposes to use inheritance to achieve this goal.
However, inheritance introduces tight coupling if the subclasses depend on implementation details of their parent class.

Instead, we use interfaces instead of superclasses to allow different implementations which you can easily substitute without changing the code that uses them.  
Interfaces are closed for modifications, and you can provide new implementations to extend the functionality of your software.

**Benefits**
##### Interface introduces an additional level of abstraction
...which enables loose coupling.   
The implementations of an interface are independent of each other.

In case two implementations of an interface share some code, you can either use inheritance or composition.

##### Example
_BasicCoffeeMachine.java_
```
import java.util.HashMap;
import java.util.Map;

public class BasicCoffeeMachine {

    private Map<CoffeeSelection, Configuration> configMap;
    private Map<CoffeeSelection, GroundCoffee>; groundCoffee;
    private BrewingUnit brewingUnit;

    public BasicCoffeeMachine(Map<CoffeeSelection, GroundCoffee> coffee) {
	this.groundCoffee = coffee;
	this.brewingUnit = new BrewingUnit();

	this.configMap = new HashMap<>();
        this.configMap.put(CoffeeSelection.FILTER_COFFEE, new Configuration(30, 480));
    }

    public Coffee brewCoffee(CoffeeSelection selection) {
	Configuration config = configMap.get(CoffeeSelection.FILTER_COFFEE);

	// get the coffee
	GroundCoffee groundCoffee = this.groundCoffee.get(CoffeeSelection.FILTER_COFFEE);

	// brew a filter coffee
	return this.brewingUnit.brew(CoffeeSelection.FILTER_COFFEE, groundCoffee, config.getQuantityWater());
    }

    public void addGroundCoffee(CoffeeSelection sel, GroundCoffee newCoffee) throws CoffeeException {
		GroundCoffee existingCoffee = this.groundCoffee.get(sel);
		if (existingCoffee != null) {
			if (existingCoffee.getName().equals(newCoffee.getName())) {
				existingCoffee.setQuantity(existingCoffee.getQuantity() + newCoffee.getQuantity());
			} else {
			throw new CoffeeException("Only one kind of coffee supported for each CoffeeSelection.");
			}
		} else {
				this.groundCoffee.put(sel, newCoffee);
		}
    }
}
```
_BasicCoffeeApp.java_
```
public class BasicCoffeeApp {

    private BasicCoffeeMachine coffeeMachine;

    public BasicCoffeeApp(BasicCoffeeMachine coffeeMachine) {
	this.coffeeMachine = coffeeMachine;
    }

    public Coffee prepareCoffee(CoffeeSelection selection) throws CoffeeException {
	Coffee coffee = this.coffeeMachine.brewCoffee(selection);
	System.out.println("Coffee is ready!");
	return coffee;
    }

    public static void main(String[] args) {
	// create a Map of available coffee beans
    	Map<CoffeeSelection, GroundCoffee> beans = new HashMap<CoffeeSelection, GroundCoffee>();
    	beans.put(CoffeeSelection.FILTER_COFFEE, new GroundCoffee(
    	    "My favorite filter coffee bean", 1000));

    	// get a new CoffeeMachine object
    	BasicCoffeeMachine machine = new BasicCoffeeMachine(beans);

    	// Instantiate CoffeeApp
    	BasicCoffeeApp app = new BasicCoffeeApp(machine);

    	// brew a fresh coffee
    	try {
    	    app.prepareCoffee(CoffeeSelection.FILTER_COFFEE);
    	} catch (CoffeeException e) {
    	    e.printStackTrace();
    	}
    }
}
```

But when your BasicCoffeeMachine got replaced, our CoffeeApp doesn't support this kind of coffee machine.  
This is where the Open/Closed principle comes in handy.

Following the Open/Closed Principle, first, you need to extract an interface that enables you to control the coffee machine. e.g. _CoffeeMachine_ Interface that has a _brewCoffee_ method.  
All classes that implement _CoffeeMachine_ interface has to implement _brewCoffee_ method.

You can then add implementations that implements the _CoffeeMachine_ interface, such as _BasicCoffeeMachine_ or _PremiumCoffeeMachine_.

Then, you have to change _CoffeeApp_ class to use the interface. You need to instantiate a specific _CoffeeMachine_ implementation in the main method.

```
import java.util.HashMap;
import java.util.Map;

public class CoffeeApp {

    private CoffeeMachine coffeeMachine;

    public CoffeeApp(CoffeeMachine coffeeMachine) {
	this.coffeeMachine = coffeeMachine;
    }

    public Coffee prepareCoffee(CoffeeSelection selection) throws CoffeeException {
	Coffee coffee = this.coffeeMachine.brewCoffee(selection);
	System.out.println("Coffee is ready!");
	return coffee;
    }

    public static void main(String[] args) {
	// create a Map of available coffee beans
    	Map<CoffeeSelection, CoffeeBean>; beans = new HashMap<CoffeeSelection, CoffeeBean>();
    	beans.put(CoffeeSelection.ESPRESSO, new CoffeeBean(
    	    "My favorite espresso bean", 1000));
    	beans.put(CoffeeSelection.FILTER_COFFEE, new CoffeeBean(
    	    "My favorite filter coffee bean", 1000));

    	// get a new CoffeeMachine object
    	PremiumCoffeeMachine machine = new PremiumCoffeeMachine(beans);

    	// Instantiate CoffeeApp
    	CoffeeApp app = new CoffeeApp(machine);

    	// brew a fresh coffee
    	try {
    	    app.prepareCoffee(CoffeeSelection.ESPRESSO);
    	} catch (CoffeeException e) {
    	    e.printStackTrace();
    	}
    } // end main
}
```

**In conclusion**
Open/Closed principle promotes the use of interfaces to enable you to adapt the functionality of your application without changing the existing code.

#### Liskov Substitution Principle (LSP)

It expends the Open/Closed Principle by focusing on the behavior of a superclass and its subtypes.

Formally,  
**>> Objects should be replaceable with instances of their subtypes without altering the correctness of that program.**

An overridden method of a subclass needs to accept the same input parameter values as the method of the superclass. This means you can implement less restrictive validation rules, but not enforce stricter rules in your subclass.

Returning to the _CoffeeMachine_ problem, assume you want to add two methods: _addCoffee_ and _brewCoffee_.

The _BasicCoffeeMachine_ can only brew filter coffee, so the _brewCoffee_ method checks if the provided _CoffeeSelection_ is FILTER_COFFEE before it calls the private method _brewFilterCoffee_.  
The _addCoffee_ method expects a _CoffeeSelection_ enum value and a _GroundCoffee_ object.

The _PremiumCoffeeMachine_ expects an object of type _CoffeeBean_ instead of _GroundCoffee_.

To solve this, you can either create another abstraction, e.g., _Coffee_, as the superclass of _CoffeeBean_ and _GroundCoffee_ and use it as the type of the method parameter.

Another approach is to exclude the _addCoffee_ method from the interface or superclass since you can't interchangeably implement it.

**In conclusion**
 * Don't implement any stricter validation rules on input parameters than implemented by the parent class.
 * Apply at least the same rules to all output parameters as applied by the parent class.

#### Interface Segregation Principle (ISP)
**>> Clients should not be forced to depend upon interfaces they do not use.**

##### Origin
The design problem was that a single Job class was used by almost all of the tasks. Whenever a print job or a stapling job needed to be performed, a call was made to the Job class.  
This resulted in a 'fat' class with multitudes of methods specific to a variety of different clients. Because of this design, a staple job would know about all the methods of the print job, even though there was no use for them.

The goal of the ISP is to reduce the side effects and frequency of required changes by splitting the software into multiple, independent parts.

1. The new coffee machine is a _FilterCoffeeMachine_ or an _EspressoCoffeeMachine_. In this case, you only need to implement the corresponding interface.

2. The new coffee machine brews filter coffee and espresso. This situation is similar to the first one. The only difference is that your class now implements both interfaces; the _FilterCoffeeMachine_ and the _EspressoCoffeeMachine_.

3. The new coffee machine is completely different to the other two. Maybe it’s one of these pad machines that you can also use to make tea or other hot drinks.  
In this case, you need to create a new interface and decide if you want to extend the _CoffeeMachine_ interface.  
In the example of the pad machine, you shouldn’t do that because you can’t add ground coffee to a pad machine. So, your _PadMachine_ class shouldn’t need to implement an _addGroundCoffee_ method.  

4. The new coffee machine provides new functionality, but you can also use it to brew a filter coffee or an espresso. In that case, you should define a new interface for the new functionality.  
Your implementation class can then implement this new interface and one or more of the existing interfaces. But please make sure to segregate the new interface from the existing ones, as you did for the _FilterCoffeeMachine_ and the _EspressoCoffeeMachine_ interfaces.

#### Dependency Inversion Principle (DIP)
**>> High-level modules, which provide complex logic, should be easily reusable and unaffected by changes in low-level modules, which provide utility features.**

To achieve that, you need to introduce an abstraction that decouples the high-level and low-level modules from each other.

The DIP consists of two parts:
1. High-level modules should not depend on low-level modules. **Both** should depend on abstractions.
2. Abstractions should not depend on details. Details should depend on abstractions.

If you continuously apply Open/Closed Principle and Liskov Substitution Principle to your code, it will then also follow the Dependency Inversion Principle.
It enables you to change higher-level and lower-level components without affecting any other classes, as long as you don’t change any interface abstractions.
