---
title: Ejb3_Tutorial_3_First_EJB
---
Here is a list of the classes we'll convert to Session Beans:
* BookDao
* Library
* LoanDao
* PatronDao
* ResourceDao

What about naming conventions? Every Session Bean has at least one interface and one class. We need to pick a name for the interface and the class. One convention is to add "Bean" after the name of the interface. So if we had an interface called RepairFacility, then the implementation would be called RepairFacilityBean. We'll use this convention and end up with the following names:

| Original| Interface| Bean|
|BookDao|BookDao|BookDaoBean|
|Library|Library|LibraryBean|
|LoanDao|LoanDao|LoanDaoBean|
|PatronDao|PatronDao|PatronDaoBean|
|ResourceDao|ResourceDao|ResourceDaoBean|

We are going to have to rename each of our beans and then create an interface. Luckily Eclipse will take care of most of this for us.

### Rename

We'll start with PatronDao. To Rename it:
* Select PatronDao in the Package Explorer
* Right-click, select **Refactor:Rename**
* Enter **PatronDaoBean** for the name
* Press **OK**

### Make it a Stateless Session Bean

We need to annotate this class to declare it is a session bean. Annotate the class with @Stateless.

### Extract Interface

Next, we need to extract the interface automatically:
* Select PatronDaoBean in the Package Explorer
* Right-click, select **Refactor:Extract Interface**
* Enter PatronDao for the interface name
* Select all of the methods
* Make sure to select **Use the extracted interface type where possible**
* Optionally deselect **Declare interface methods 'public'**
* Optionally deselect **Declare interface methods 'abstract'**
* Click **OK**

### Update Unit Test: PatronDaoTest

The unit test inherits from BaseDbDaoTest. This adds support for creating dao's entity managers, etc. We don't want to do any of this, so we can safely remove the base class.

We need to updated getDao() in the following ways:
* It no longer should have the @Override annotation
* It simply uses the JBossUtil to look-up the dao
* Make sure to remove the dao instance variable and fix any resulting compilation errors by replacing **dao** with **getDao()**

Here is an updated version of that method:

{% highlight java %}
    public PatronDao getDao() {
        return JBossUtil.lookup(PatronDao.class, "PatronDaoBean/local");
    }
{% endhighlight %}

Notice two things about the name we provide. First, we use the unqualified name of the bean class, **PatronDaoBean**. Also notice we need to add **/local**. If you had a remote interface, you'd instead use **/Remote**. And if you leave this off, you'll get a bad cast exception. You might experiment with this to discover why.

We also need to initialize the EJB Container. Add the following method:

{% highlight java %}
    @BeforeClass
    public static void initContainer() {
        JBossUtil.startDeployer();
    }
{% endhighlight %}

Finally, since we are now testing PatronDaoBean instead of PatronDao, we might want to rename the test to PatronDaoBeanTest.java.

### Run the PatronDaoBeanTest Tests: First Failure

When you run the unit tests (just PatronDaoBeanTest), they will all fail. If you review the stack trace in the JUnit window, you'll notice that all the lines that fail look something like this:

{% highlight java %}
        getEm().persist(p);
{% endhighlight %}

We're getting a null pointer exception on this line because getEm() returns null. There was a method with the @Before annotation that set the entity manager on the PatronDao. We no longer inherit from that class so we no longer get that initialization. However, this is not how we should be initializing that attribute anyway. We can use the container to perform this initialization. 

We can have the container inject the entity manager (or an entity manager factory if you'd like) into our PatronDaoBean.

To get an entity manager injected, we use the annotation @PersistenceContext on an attribute of type EntityManager. Since we inherit the entity manager attribute from a base class, we place that annotation in the base class, BaseDao, as follows:

{% highlight java %}
public abstract class BaseDao {
    @PersistenceContext(unitName = "lis")
    private EntityManager em;
    ...
}
{% endhighlight %}

Using the @PersistenceContext will tell the container to look up the named EntityManagerFactory, create an EntityManager for us and then place that entity manager into the variable, in this case em. This happens when we look up the PatronDao.

Execute the PatronDaoBeanTest tests only -- if you were to run all the tests in the project you would get misleading errors at this point ("could not insert [entity.Address]").

### Second Attempt: Second Failure

After adding in @PersistenceContext we get one test to pass and three to fail. If you look at the stack trace in the JUnit window, we see that the Patron class does not have a default constructor. We need to add a default constructor to Patron.java:

{% highlight java %}
    public Patron() {
    }
{% endhighlight %}

Add it and re-run the tests (again, just PatronDaoBeanTest).

### Go back and do the steps you just did for all the dao classes



### Success
This fixes all the tests in PatronDaoBeanTests. For some reason when we ran this code in a JSE environment, it "worked" even though it did not comply with the standard. Overall this first conversion was fairly painless. However, before we go on, are out tests isolated? That is, after we execute the tests did we remember to remove everything we created?

### Review: Are We Isolated?

At this point we need a tool to review the contents of the data base. If you want to work directly in Eclipse, you can use Quantum DB. If you prefer to work outside of Eclipse (and frankly with a more powerful tool), then you might want to try SQuirrel Sql Client.

If you start with a clean database and run the tests, 2 patrons are left in the database after the tests execute.

This means that the tests leave a foot print. Or they are not isolated. We want our tests to have no side-effects because they might run in any order and such side effects could cause other tests to fail. In the old way of doing things, we started a transaction and then rolled it back, so nothing got saved to the database. We have three options on how to avoid this:

* Try to simulate the old behavior
* Create a new database every time
* Clean up after ourselves

The first option is tricky at best. We've already seen that we missed some things in a JSE environment and, more importantly, this is not how our system will be running so even if we the first option to work, we've not really improved anything.

The second option is good but it has a few flaws:
* It simply masks bad tests and unless we drop the database after **every** test, we have not solved any problem.
* What if we want to run our tests against a populated database? We could re-create the database, but we'd still have to do it after **every** test.

Really, the best option is to write our tests so they clean up after themselves.

There are 4 tests. One is to test removing a Patron so it runs clean. Another looks up a Patron with a bogus key, so it does not have any side effects. This leaves the following two tests we need to fix:
* createAPatron
* updateAPatron

Here are the updates to PatronDaoBeanTest:

{% highlight java %}
    @Test
    public void createAPatron() {
        final Patron p = createAPatronImpl();

        try {
            final Patron found = getDao().retrieve(p.getId());
            assertNotNull(found);
        } finally {
            removePatron(p);
        }
    }

    @Test
    public void updateAPatron() {
        final Patron p = createAPatronImpl();

        try {
            final String originalPhoneNumber = p.getPhoneNumber();
            p.setPhoneNumber(NEW_PN);
            getDao().update(p);
            final Patron found = getDao().retrieve(p.getId());

            assertNotNull(found);
            assertFalse(NEW_PN.equals(originalPhoneNumber));
            assertEquals(NEW_PN, p.getPhoneNumber());
        } finally {
            removePatron(p);
        }
    }

    private void removePatron(final Patron p) {
        getDao().removePatron(p.getId());
    }
{% endhighlight %}

We used a try {} finally block to make sure after the test finishes that we call a support method, removePatron. The fix is trivial (so far). We also added a simple private method to actually perform the delete.
