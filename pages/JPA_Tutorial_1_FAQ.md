---
title: JPA_Tutorial_1_FAQ
---
## JPA Tutorial 1 - FAQ
 * **What's the entity Manager?**  Object cache which persists entities.  Manages any objects with the @Entity annotation.  Allows starting/stopping transactions, crud'ing database.  "Your facade to working with the database"
* **Is hibernate using an in-memory database?**  Hibernate is one implementation of JPA.  It connects to the database through a jdbc url.  The database we use in the early exercises is Hypersonic SQL.
* **When do you need a default constructor?**  Spec says you always need it.  If there are no other constructors, java provides the default constructor.  If you add a single constructor (w/parms), keep in mind the default constructor is removed by java. 
* **Annotations?**  Annotations are type-safe meta-data about a class.  They can target fields/methods/classes.  Reflection allows you to query the state of any annotation.  Annotations have 'lifetimes' (compile-time, load-time, run-time).
* **Compared to toplink:  How are isolation levels handled? What kind of caching concerns do we have to worry about?**  There will be similiar challenges in tuning.
* **JEE source?**  Download from JBoss (svn) or get from Brett.
* **Where does @Id go?**  (method or field?) If it's on an attribute, JPA will only look for annotations on fields, and use reflection to access fields (without getter's and setter's).  If it's on a method (getId()), JPA will only look for annotations on methods, and will use getter's and setter's to access fields.
* **Is there a reason @id is not private?**  Brett forgets things as he gets older. (It should be private)
* **When creating the job class, do we need equals() and hashcode()? **  not yet
* **In Company.java there is a lazy-initialization in getEmployees() .  Why?**  When you retrieve from the db, hibernate creates an object with newInstance (reflection).  Lazily initializing the employees collections could decrease unnecessary garbase collection when retrieving large result sets.  This is just a consideration (not always the 'best' way).
* **What's with the equals() method on Person?**  You don't want an equals() who's value changes over time.  (pre-insert, the id might be null, post-insert, it would have a value)
* **How many times does it go to the database when you have a one-to-many and you retrieve the 'one' side. **   By default, once for the root ('one') object, and then one or more times (up to the container's discretion) to retrieve the 'many' side -- as you iterate through the collection.  You can override with fetch=FetchType.EAGER (to fetch with a join and retrieve everything up front).  See the actual sql with hibernate.show_sql=true. 
* **Can you turn off the 'caching' part of JPA?**You can define transactional boundries to limit the scope of the cache.  Look up 'report queries' for some examples. 
* **How do you know what you set 'mappedBy' to? ** Use the name of the field that defines the relationship 
* **What happens if you manipulate your objects outside of a transaction? ** In a JSE environment, objects are still managed by the EntityManager. So if you make a change to an object, then later start a transaction and commit it, the changes made outside of the transaction are also sent to the database. If, on the other hand, you close the EntityManager or flush it, the changes are lost.

### Brett: What have you learned in this tutorial? 
### Class responses:

* Eclipse is good
* JPA hides a lot of JDBC complexity. 
* How to define relationships of entities
* Using entity notations


### Responses to questions in the exercises 
* **What does @Entity do? **

* defines a class as being an entity bean.
* Tells container what behavior to add through bytecode instrumentation or dynamic proxies

* **What does the @Id do ? **

* defines primary key
* required for entities
* only @Id fields can be passed into the em.find(class, id) method

* **What does the 'generatedValue' do ? **

* automatically assigns id based on the defined strategy

* **What's embeddable vs. embedded? **

* Defines how the data is stored in the db.  (serialization vs fields being persisted to columns)



## Day 1 Review 
### What we learned today . . .
* JUnit
* many-to-one & vice versa (bi-directional)
* Annotations & JUnit
* Eclipse
* Entity Managers
* hibernate
* refactoring
* eclipse shortcuts, embeddable stuff, understanding the relationships
* eclipse shortcuts (!)
* identifying annotation and attributes/properties
* static imports
* persisting all the entities (without using cascade attributes)

### feedback so far on the class format
* "better than just hearing somebody talk"
* hands-on is good, maybe should have an overview at the beginning though.  "pedagogical"
 
### What we'll do tommorrow
* more tdd style approach
* tutorial 3 (big project)
* polymorphic queries
