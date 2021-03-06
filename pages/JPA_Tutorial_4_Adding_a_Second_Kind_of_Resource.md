---
title: JPA_Tutorial_4_Adding_a_Second_Kind_of_Resource
---
Considering the amount of work required to get ready to add a resource, you might think that creating a hierarchy and then performing so-called polymorphic queries is difficult. It isn't. The difficulty of moving towards supporting a hierarchy is that we started with a concrete class and wanted to later introduce an abstract concept, the Resource.

In this final step for the tutorial, we'll create a second kind of resource, a DVD and then write some basic tests.

### Supporting DVD's
The first basic test is creating a dvd and checking it out to a Patron. This is a test we add to LibraryTest.java.

### First test: Checkout a Dvd
{% highlight java %}
    @Test
    public void checkoutDvd() {
        final Patron p = createPatron();
        final Dvd d = createDvd();
        library.checkout(p.getId(), CURRENT_DATE, d.getId());
        assertTrue(d.isOnLoanTo(p));
        assertEquals(CURRENT_PLUS_6, d.getDueDate());
    }

    private Dvd createDvd() {
        return library.createDvd("Raiders of the Lost Ark", new Name("Steven",
                "Spielberg"));
    }
{% endhighlight %}
This test uses a new constant, **CURRENT_PLUS_6**, here's the change to LibraryTest to support that.
{% highlight java %}
public class LibraryTest extends EntityManagerBasedTest {
    private static Date CURRENT_PLUS_6; // add this attribute

    // update the setupDates method
    @BeforeClass
    public static void setupDates() {
        Calendar c = Calendar.getInstance();
        DateTimeUtil.removeTimeFrom(c);
        CURRENT_DATE = c.getTime();
        c.add(Calendar.DAY_OF_MONTH, 6);
        CURRENT_PLUS_6 = c.getTime();
        c.add(Calendar.DAY_OF_MONTH, 2);
        CURRENT_PLUS_8 = c.getTime();
        c.add(Calendar.DAY_OF_MONTH, 6);
        CURRENT_PLUS_14 = c.getTime();
        c.add(Calendar.DAY_OF_MONTH, 1);
        CURRENT_PLUS_15 = c.getTime();
    }
{% endhighlight %}
To get this to compile, we need to add support in the library class for creating a DVD. Here's that basic support:

### Updating Library
{% highlight java %}
    public Dvd createDvd(final String title, final Name directorsName) {
        final Director director = new Director();
        director.setName(directorsName);
        final Dvd d = new Dvd();
        d.setTitle(title);
        d.setDirector(director);
        getResourceDao().create(d);
        return d;
    }
{% endhighlight %}

Notice that we have a director and a dvd in this example. Here are those classes:
### Director.java
{% highlight java %}
package entity;

import javax.persistence.Embedded;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Director {
    @Id
    @GeneratedValue
    private Long id;

    @Embedded
    private Name name;

    public Director() {
    }

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public Name getName() {
        return name;
    }

    public void setName(Name name) {
        this.name = name;
    }
}
{% endhighlight %}

### Dvd.java
{% highlight java %}
package entity;

import java.util.Calendar;
import java.util.Date;

import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.ManyToOne;

/**
 * Notice that to get this to "work" all we need to do is inherit from Resource.
 * As with book, we do not define an id field, since it is inherited.
 * 
 * The way this class is represented in the database is determined by meta
 * information in the Resource base class.
 * 
 * Of course, regular Java rules apply here. The Resource base class defines 2
 * abstract methods, both of which are overridden here.
 */
@Entity
public class Dvd extends Resource {
    @ManyToOne(cascade = { CascadeType.MERGE, CascadeType.PERSIST })
    private Director director;

    @Override
    public Date calculateDueDateFrom(final Date checkoutDate) {
        final Calendar c = Calendar.getInstance();
        c.setTime(checkoutDate);
        c.add(Calendar.DATE, 6);
        return c.getTime();
    }

    @Override
    public double calculateFine(final int daysLate) {
        if (daysLate < 5) {
            return .3 * daysLate;
        }

        if (daysLate < 10) {
            return .6 * daysLate;
        }

        return 6.0;
    }

    public Director getDirector() {
        return director;
    }

    public void setDirector(Director director) {
        this.director = director;
    }
}
{% endhighlight %}

### Resource.java
To get this to actually work, we need to use the @Inheritance annotation on resource. Here's the change:
{% highlight java %}
//...
import javax.persistence.Inheritance;
import javax.persistence.InheritanceType;
@Entity
@Inheritance(strategy = InheritanceType.JOINED)
public abstract class Resource {
    //...
}
{% endhighlight %}

As mentioned before, there are three kinds of representations for hierarchy. We are using the so-called JOINED approach, which means there is a table per class. It turns out that we need to use this approach based on how we've defined Book. Why that is will be answered in a following assignment.

### A Few More Tests
Here are a few more tests to verify basic functionality. These also belong in LibraryTest.java.
{% highlight java %}
    @Test
    public void returnDvdLate() {
        final Dvd d = createDvd();
        final Patron p = createPatron();

        library.checkout(p.getId(), CURRENT_DATE, d.getId());
        library.returnResource(CURRENT_PLUS_8, d.getId());

        assertEquals(1, p.getFines().size());
        assertEquals(0.6d, p.calculateTotalFines());
    }
    
    @Test
    public void checkoutDvdAndBook() {
        final Dvd d = createDvd();
        final Book b = createBook();
        final Patron p = createPatron();
        
        library.checkout(p.getId(), CURRENT_DATE, d.getId());
        library.checkout(p.getId(), CURRENT_DATE, b.getId());
        
        final List<Resource> list = library.listResourcesOnLoanTo(p.getId());
        assertEquals(2, list.size());
        assertTrue(d.isOnLoanTo(p));
        assertTrue(b.isOnLoanTo(p));
    }
{% endhighlight %}

### Summary: Inheritance
That's pretty much it. If we had started with a base class called Resource, this work would have been trivial. Adding new subclasses is quite easy. Figuring out which representation may take a little experimentation, but it's easy to change.

Queries are inherently polymorphic in nature if that makes sense for the given query. Your challenge is working with a base type insead of down-casting to a derived type.
