---
title: 7d6fb807ed70179110584fc0291a1499f3fd1e09
---
These are terms that often come up in classes I teach and I often use the same examples for certain terms, so I'm starting to capture them and write them down.

## Concepts Related to OO Terminology

----

### Class
A blueprint for objects. A class describes the structure of an object as well as the behavior of an object. The structure is described in terms of the data each object will have. The behavior in terms of the methods. Here are some additional metaphors:
* A class is factory that produces only one product
* A class is like an item in a catalog, it describes the item but it is not the item. for example, if you see a pair a jeans in a catalog, you can't wear them. You have to order (instantiate) them. Then when you get them, they take on a life of their own. They have their own fade lines, stains and holes, but they are essentially still jeans.

----

### Object
An object is, typically, the thing that gets work done. An object-oriented system comprises several objects of different classes interacting to solve a problem. An object has its own data (as described by its class). An object supports some behavior (as described by its class). An object either implicitly or explicitly knows its class.

Let's take the "item in a catalog" metaphor a bit further. I ordered a pair of jeans based on the description in a catalog. The description included several choices I had to make when ordering, including: color, waist size, length. The description limited my choices. That is, the "class" of jeans was limited in terms of what colors were available, as well as waist sizes and lengths. The "class" also described the number of pockets, the cut, etc.

Now that I have my jeans, I see that my jeans are blue, 36" waist, and 30" length. I could have ordered 2 pairs. If so, each would be similar but each would have its own life. If I happen to accidentally wash a pair of jeans with bleach, that pair's color will be somewhat different (probably) than the one that did not get bleached. Each instance of jeans is similar in structure but unique in history.

An object takes up memory. Technically so does a class. The amount of memory a class takes up is related to how much code is associated with the class. The size an individual object takes up in memory is related to how much data it holds. Typically, a system will have multiple objects of the same class at the same time in memory, so typically objects take up more memory than classes.

----

### Instantiation
The act of creating an object from a class. I ask the class to create an instance. Some memory gets allocated. The object gets initialized and then I can use the object.

When I ordered my pair of jeans, eventually I got the jeans in the mail. That's the equivalent of instantiation. Notice I don't really know if they were made to order if they already existed. That's a possibility as well.

----

### Polymorphism
Quick answer: Same message different method.

Imagine I have written a Monopoly program. In monopoly we have several classes: Player, Board Location, Dice, etc. If I actually look a bit deeper, not all BoardLocations are the same. I have some BoardLocations that I can own (Real Estate). I have some BoardLocations that give me cards (Community Chest and Chance). There's Go, Go To Jail and Luxury Tax, just to name a few.

When I land on a board location, what happens?
* If I **land on** Go, I receive $200
* If I **land on** Go To Jail, I go directly to Jail (and don't get $200)
* If I **land on** Reading Railroad, what happens depends on if it is owned or not
* If I **land on** Community Chest, I get a card and then something happens there

Here's an example hierarchy:
![](images/PartialBoardLocationHierarchy.gif)

Imagine I have different kinds of Board Locations (read this as different classes). I have a class for Go To Jail, Go, Railroads, Community Chest, Luxury Tax, Jail, Utilities, etc. Each of these classes responds to the message **land on**. However, each one responds a bit differently.

So I have several **different** classes. Each of these class has a **method** called **landOn**. Each of these **landOn** methods has different code. So when I send the landOn **message** to an instance of the Go class, I get the **method** that is in the go class, which gives me $200. When I send the landOn **message** to an instance of the Luxury Tax class, I get the **method** in associated with with the Luxury Tax class, and I get charged a fine (my choice, $200 or 10% of what I own).

Same **message**: landOn
Different **method**: landOn in the Go class, landOn in the Jail tax, landOn in the Utility class, etc.

Two classes do not have to be related necessarily. For example, when I send the **roll** message to dice, I get a value from 2 to 12. When I send the **roll** message to a virtual dog, it will do a trick.

Since an object knows its class, when I send a message to an object, the code that executes is based on the object(receiver) of the messages (i.e. based on the class of the receiver).

----

### Inheritance
**Quick Reminder:** Inheritance is often overused. Prefer delegation to inheritance.

I can define a class and then create other classes that take on all of the characteristics of the first class. Characteristics include both behavior (methods) and structure (attributes). The first class is called the parent or base class, the second class is called the child or derived class. The parent class along with all of its children classes is called an inheritance hierarchy or often just hierarchy.

A few guidelines: Inheritance is often overused. Take the following into consideration:
* Make sure the child **is a** parent. That is, anywhere you can use the parent you can also use the child.
* A given hierarchy should address one concern (the reason for the hierarchy should be cohesive). Do not, for example, use the same hierarchy for both Graphical User Interface(GUI) Components as well as Platform Variations. Imagine a base class, Window Element, with several sub classes: Box, Dialog, Menu, etc. Then you introduce a platform layer so you get a Unix Box, Unix Dialog, Unix Menu. Windows Box, windows Dialog, Windows Menu. Mac Box, Mac Dialog, Mac Menu. Instead have one hierarchy for GUI Components and one for Platforms. Then you can "plug in" the platform to the GUI component. Notice that in the first example we had a total of 13 classes while in the second we had 8. The second form grows much slower than the first and is much less brittle. Separating concerns in a hierarchy is an example of the more general [Fundamental_Terms_With_Examples#SquareLawOfComputation](Fundamental_Terms_With_Examples#SquareLawOfComputation).
* Prefer delegation to inheritance.
* Avoid violating the [Liskov Substitution Principle](http://www.objectmentor.com/resources/articles/lsp.pdf). That is, when considering using a class as a parent for another, make sure that everything that applies to the parent class applies to the child class now and into the future (this should make inheritance harder to justify).

----

### Cohesion
**Quick Reminder:** Try to maximize cohesion and reduce unnecessary coupling.

Something that does one thing or a few highly-related things well is cohesive. Something that does many less-related things is not-cohesive. 

#### Example 1
In the inheritance example, we described two example hierarchies:
![](images/GuiPlatformHierarchy.gif)

![](images/GuiHasAPlatform.gif)

In the first diagram we couple the notion and Gui Component and Platform into on hierarchy. In the second, we separate those concerns into two hierarchies.

What happens if we add a new platform? In the first example we have to add one new class for each kind of Gui Component (currently 3, but you can imagine there would be more like 20+). In the second example, we add one class.

#### Example 2
Imagine I have a database that stores Vehicles. I need to create, read, update and delete vehicles. Before I do so I need to validate that the vehicles are well-defined. So I have two basic kinds of requirements:
* Manage rows in a database
* Perform data validation

If I choose to write this using code in a general programming language (versus doing everything in the database for example), then I have a few options:
* One class that does both things.
* One class that does database stuff, one class that does validation.
* One class for Create, one class for Read, one class for Update, one class for Delete, one class for Validation.

The first option is less cohesive than the second. One class does what might be considered two sets of things (I say might be considered because different people looking at those requirements might not agree that they are really different, it's a matter of perspective). I think the third option is probably a bit too split up. Why? Because there's a lot of commonality behind what you need to do when Creating, Reading, Updating and Deleting so putting those together offers some advantages. However, it's definitely bad (neither is the first option).

#### Why do you care?
Sometimes we don't care. For example, in a home stereo system with multiple components (e.g. DVD Player, Amplifier, Pre-Amp, Speakers, Cassette Player), the individual components are cohesive. A boom box does not share the same level of cohesion. Which is better?

Neither, they server different purposes. Sometimes doing lots of things in one package is exactly what you want. However, there are some costs (in both situations):
* Home system uses more power, each component has its own power supply and power cord.
* Home system probably sounds better and louder.
* If the DVD player breaks in a home system, you can get it fixed while using the other components. The boom box does not support that.
* You cannot take a home system to the beach.

In software, we care for several reasons:
* Putting more things in one place makes the code harder to write, read and maintain.
* If your solution is not cohesive overall, a simple requirement change might tend to ripple more. You change something for one requirement in a class that also addresses other requirements and you break something with an unrelated requirement because of a lack of cohesion.

----

### Coupling
**Quick Reminder:** Try to minimize coupling.

In a home stereo system with multiple components (e.g. DVD Player, Amplifier, Pre-Amp, Speakers, Cassette Player), the individual components are loosely coupled. E.g. the DVD player has no connection to the Cassette Player. Where the is coupling, the DVD player connected to the AMP, the coupling is loose and well defined (three RCA cables, or S-VGA and 2 RCA Cables, etc.). The cassette player hooks up with the same connectors, it just uses less.

That is, there is a well-defined interface between the AMP and various components. As long as a given component conforms to that interface, you can hook it up. If it does not (e.g. a turn table), you need a special connector or an adapter to get it to work.

Contrast that with a boom box. The components are physically coupled (in a plastic box) and electrically coupled (all use the same power supply).

In software, the more coupled your classes are, the more difficult it will be to add functionality without breaking seeming unrelated things.

On the other hand, some level of coupling is required. If there is no coupling, then there's no system.

----

### Delegation
You ask me to do something. Instead of doing it myself, I have someone else do it. You don't know how it got done, just that you asked me to do it.

Consider, again, Monopoly. In Monopoly I have two kinds of locations where something happens when I land on them, Community Chest and Chance. When you

----

### Encapsulation
What did you have for breakfast? The real question is how do I get answer to that question. Understand that and you'll get a better understanding of how people tend to use the word encapsulation. So how could I answer that question:
* I could pump your stomach, analyze the contents a la CSI and probably get a good idea of what you had (so long as it has not been too long).
* I could ask you.

Which do you prefer? My guess is that our objects have the same preference. In the first approach I am violating your encapsulation and, frankly, not getting good information. With the second option, I defer to you. You could lie, but in the object world I'm not too concerned about that.

What we are discussing is information hiding. If I want to know something about an object, I'll ask it. I won't directly look at all of its innards (attributes).

----

### Method
A method is code written in a class. For example, landOn() is a message I can send to any kind of Board Location.

----

### Message
A request to an object to do something. A message causes the execution of a method. Why are there two terms? Imagine I have a Player class that plays Monopoly. That player takes a turn and ultimately landsOn a board location. In the "real world" a Player knows the kind of location s/he landed on. In the software world it does not. It knows that it landed on a Board Location. It tells that board location "landOn" (that is, it sends the message). The particular Board Location, knowing its class, makes sure to execute the appropriate method. 

This is an example of **polymorphism** as well.

----
----

## Laws

### Rule of 3
If you cannot list at least three problems with your understanding of a problem, you don't understand the problem - Weinberg, Are Your Lights On?

Variations:
* **Every** solution introduces problems. If you cannot list three problems your solution will introduce, you don't understand your solution (or the problem).

----

{: #SquareLawOfComputation}
### Square Law of Computation
The complexity to solve a problem grows at least as fast as the square of the number of things you are trying to solve - Weinberg, Introduction to General Systems Thinking

#### Example 1
If I try to solve 4 things at once, versus solving each of those 4 things individually, the relative complexity is 4x4 16 versus 4.

#### Example
When we had one hierarchy to represent both Gui Components and Platforms versus two hierarchies. The complexity of adding a new Platform in the first example was equal to N (where N is the number of kinds of Gui components, probably 20+). In the second case it was 1.

Often we strive to try to separate our problems so we can solve them individually. We call this divide and conquer. The Square Law of Computation
