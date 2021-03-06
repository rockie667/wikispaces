---
title: JPA_Tutorial_3_FAQ
---
JPA Tutorial 3 - FAQ
* **Phone number length is set to 11 in the annotation in the exercise, but phone numbers are 12. What's happening?**It will get truncated (in hypersonic)//
* **TDD: Is there a tool (eclipse plugin) to generate a concrete class based on a test case?**Use Quick-fix (Ctrl-1 or click the icon on the sidebar) to create the class, and stub methods.//
* **When do Named Queries get compiled?**When you create the EntityManager factory.//
* **Is there something that can use to see the db structure based on the mappings?**After the entitymanager creates the db, use something like DB Visualizer//

### What did we learn/observe so far (mid-day)?
* varargs for methods
* cascading types - persist, all, refresh. propagates operations (merge/update/persist/refresh) across entities. Saves you from having to persist things.
* jpa requires jdk 1.5
* the @column annotation sets the name of the column in the database. also lets you specify things like width/length (attribute of the annotation).
* debugging good
* eclipse can gen hashcode/equals

### What did we learn/observe so far (end of day)?
* Brett: Why we use a DAO .
* Brett: Contrast the Library to the DAO's .
* ? When you get your objects, they're fully configured (nice). Brett: makes it more testable
* ? Defines attributes for a particular field in DB. Brett: affects physical characteristics as opposed to logical.
* ? saves lines of code
* ? brett: hide a warning based on actual query parameter type (not available at compile-time)
* ? You can have only one version of the annotation on a given object
* ? confusing, brett: heavy-handed/overkill
* ? helps if you decide you want to change how you're accessing your database.
* ? have to look at code to understand what's changing (head nods)
* ) gives compiler ability to flag problem. some class discussion of when needed, when not...
* l Brett:time/datastamp

//What else have you learned today?//
* e
* )
* .
* ? defines the sort order

//Day 3 - Tutorial #3
Questions//
* **Patron has a one-to-many for fines, but no many-to-one on the fines side. Why?** Just an example of a uni-directional relationship.
* **Why is insertable false? (in exercise)** We think: So that loan has to point at things which already exist. (as opposed to creating new loan/patron at same time)
* ? Hold off on that because the next few exercises build up additional requirements which may change your mind

//What else have you learned today?//
* //How we can create/persist a given entity//
* s
* //Query syntax is better/easier//

//Notes for Brett://
* .


### s
* A Patron can return a book while having an overdue book
### Below are the tests to check everything:
And the new feature that doesn't allow to checkout with overdue books.

These tests are in LibraryTest.java
{% highlight java %}
    @Test
    public void patronHasNumberOfOverdueBooks() {
        final Patron p = createPatron();
        final Book b1 = createBook();
        final Book b2 = createBook();
        library.checkout(p.getId(), CURRENT_DATE, b1.getId());
        int numOfBooks;
        numOfBooks = library.getNumberOfOverdueBooksOfPatron(p, CURRENT_PLUS_14);
        assertEquals(0, numOfBooks);
        library.checkout(p.getId(), CURRENT_PLUS_8, b2.getId());
        numOfBooks = library.getNumberOfOverdueBooksOfPatron(p, CURRENT_PLUS_15);
        assertEquals(1, numOfBooks);
        library.returnBook(CURRENT_PLUS_8, b1.getId());
        numOfBooks = library.getNumberOfOverdueBooksOfPatron(p, CURRENT_PLUS_15);
        assertEquals(0, numOfBooks);
    }

    // My change
    @Test
    public void patronCannotCheckoutWithOverdueBooks() {
        final Patron p = createPatron();
        final Book b1 = createBook();
        library.checkout(p.getId(), CURRENT_DATE, b1.getId());

        final Book b2 = createBook();
        try {
            library.checkout(p.getId(), CURRENT_PLUS_15, b2.getId());
            fail(String.format("1. Should have thrown exception: %s", PatronHasOverdueBooks.class
                    .getName()));
        }
        catch (PatronHasOverdueBooks e) {
            assertEquals(1, e.getNumberOfOverdueBooks());
            final Book b3 = createBook();
            library.checkout(p.getId(), CURRENT_DATE, b2.getId());
            try {
                library.checkout(p.getId(), CURRENT_PLUS_15, b3.getId());
                fail(String.format("2. Should have thrown exception: %s",
                        PatronHasOverdueBooks.class.getName()));
            }
            catch (PatronHasOverdueBooks e1) {
                assertEquals(2, e1.getNumberOfOverdueBooks());
            }
            int numOfBooks = library.getNumberOfOverdueBooksOfPatron(p, CURRENT_PLUS_14);
            assertEquals(0, numOfBooks);
        }
    }

{% endhighlight %}

### And here are the changes in project:

There is a new exception class called: **PatronHasOverdueBooks**
Here it is:
{% highlight java %}
package exception;

/**
 * Thrown when a Patron attempts checkout while he has overdue books.
 */
public class PatronHasOverdueBooks extends RuntimeException {
    private static final long serialVersionUID = 7328551028997485470L;
    private final int numberOfOverdueBooks;
    public PatronHasOverdueBooks(final int numberOfOverdueBooks) {
        this.numberOfOverdueBooks = numberOfOverdueBooks;
    }
    public int getNumberOfOverdueBooks() {
        return this.numberOfOverdueBooks;
    }

}

{% endhighlight %}

In Library class, I added to the checkout method a validation for overdue books:
{% highlight asp %}
    public void checkout(final Long patronId, final Date checkoutDate, final Long... bookIds) {
        ...
        // Checking number of overdue books of the Patron
        int booksOverdue = getNumberOfOverdueBooksOfPatron(p, checkoutDate);
        if (booksOverdue != 0) {
            throw new PatronHasOverdueBooks(booksOverdue);
        }

        ...
    }

{% endhighlight %}

Also, in Library class, the new method: getNumberOfOverdueBooksOfPatron
{% highlight java %}
    public int getNumberOfOverdueBooksOfPatron(final Patron patron, final Date compareDate) {
        return getLoanDao().numberOfOverdueBookOfPatron(patron, compareDate);
    }

{% endhighlight %}

### LoanDao can return the number of overdue books of specific Patron:
{% highlight java %}
    /**
     * Return number of overdue books of this patron
     *
     * @param patron
     * @param compareDate
     * @return int - The number of overdue books
     */
    public int numberOfOverdueBookOfPatron(final Patron patron, final Date compareDate) {
        final Query query = getEm().createNamedQuery("Loan.overdueBooksOfPatron");
        query.setParameter("date", compareDate);
        query.setParameter("patron", patron);

        return query.getResultList().size();

    }

{% endhighlight %}


It uses a new named-query. One of the parameter is the Patron (not the patron's id). I wanted to see how the join works ...
{% highlight java %}
@NamedQuery(name = "Loan.overdueBooksOfPatron", query = "SELECT l.book FROM Loan l WHERE l.dueDate < :date AND l.patron = :patron")
{% endhighlight %}
