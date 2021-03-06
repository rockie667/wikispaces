---
title: JPA_Tutorial_3_Books_on_Loan
---
## Books on Loan
You can start from this point usingthe following source files: [JapTutorial3BeforeAddingLoan.jar](files/JapTutorial3BeforeAddingLoan.jar)

The relationship between Patron and Book is:
![](images/PatronToBook.gif)

We need some more information. Right now, when we check out a book, we don't know when it is due, that seems to be a minimal amount of information. Adding this information to the Patron does not seem to make sense since there's one of these dates per** Book**, not per Patron. On the other hand, adding it to book directly means we have a field that will often be blank. There's a more natural way to model this relationship:
![](images/PatronBookLoan.gif)

A loan is associated with the** association** between Patron and Book. When the association exists, the loan exists, otherwise it does not.

We are going to test our way into this.
----
## Date Support
We're going to be working with dates extensively. Here is a utility class for working with dates and its test:
### DateTimeUtil.java
{% highlight java %}
package util;

import java.util.Calendar;
import java.util.Date;
import java.util.GregorianCalendar;

/**
 * This is a simple class containing date/time utilities to avoid proliferation
 * of duplicate code through the system.
 */
public class DateTimeUtil {
    private static final int MS_IN_HOUR = 1000 * 60 * 60;
    private static final int MS_IN_Day = 24 * MS_IN_HOUR;

    /**
     * This is a class with all static methods (often called a utility class).
     * To document the fact that it should be used without first being
     * instantiated, we make the constructor private. Furthermore, some code
     * evaluation tools, such as PMD, will complain about an empty method body,
     * so we add a comment in the method body to appease such tools.
     * 
     */
    private DateTimeUtil() {
        // I'm a utility class, do not instantiate me
    }

    /**
     * Create a date with all time values set to 0.
     * 
     * @param dayOfMonth
     *            If this value is outside the valid dates in a month, the
     *            actual date will be adjusted forward or backwards accordingly
     * 
     * @param month
     *            1-based month, 1 = January
     * 
     * @param year
     * 
     * @return Date with time cleared.
     */
    public static Date createDateOn(final int dayOfMonth, final int month,
            final int year) {
        Calendar c = Calendar.getInstance();
        c.set(Calendar.DAY_OF_MONTH, dayOfMonth);
        c.set(Calendar.MONTH, month - 1);
        c.set(Calendar.YEAR, year);
        removeTimeFrom(c);
        return c.getTime();
    }

    /**
     * Remove all of the time elements from a date.
     */
    public static void removeTimeFrom(final Calendar c) {
        c.clear(Calendar.AM_PM);
        c.clear(Calendar.HOUR_OF_DAY);
        c.clear(Calendar.HOUR);
        c.clear(Calendar.MINUTE);
        c.clear(Calendar.SECOND);
        c.clear(Calendar.MILLISECOND);
    }

    /**
     * This is a simple algorithm to calculate the number of days between two
     * dates. It is not very accurate, does not take into consideration leap
     * years, etc. Do not use this in production code. It serves our purposes
     * here.
     * 
     * @param d1
     *            "from date"
     * @param d2
     *            "to date"
     * 
     * @return number of times "midnight" is crossed between these two dates,
     *         logically this is d2 - d1.
     */
    public static int daysBetween(final Date d1, final Date d2) {
        GregorianCalendar c1 = new GregorianCalendar();
        c1.setTime(d1);
        GregorianCalendar c2 = new GregorianCalendar();
        c2.setTime(d2);

        final long t1 = c1.getTimeInMillis();
        final long t2 = c2.getTimeInMillis();
        long diff = t2 - t1;

        final boolean startInDst = c1.getTimeZone().inDaylightTime(d1);
        final boolean endInDst = c2.getTimeZone().inDaylightTime(d2);

        if (startInDst && !endInDst) {
            diff -= MS_IN_HOUR;
        }
        if (!startInDst && endInDst) {
            diff += MS_IN_HOUR;
        }

        return (int) (diff / MS_IN_Day);
    }
}
{% endhighlight %}

### DateTimeUtilTest.java
{% highlight java %}
package util;

import static org.junit.Assert.assertEquals;

import java.util.Calendar;
import java.util.Date;

import org.junit.Test;

/**
 * A class to test the DateTimeUtil class. Verifies that the calculation for the
 * number of days between to dates is correct for several different scenarios.
 */
public class DateTimeUtilTest {
    public static final Date DATE = Calendar.getInstance().getTime();

    @Test
    public void dateBetween0() {
        assertEquals(0, DateTimeUtil.daysBetween(DATE, DATE));
    }

    @Test
    public void dateBetween1() {
        assertEquals(1, DateTimeUtil.daysBetween(DATE, addDaysToDate(DATE, 1)));
    }

    @Test
    public void dateBetweenMinus1() {
        assertEquals(-1, DateTimeUtil
                .daysBetween(DATE, addDaysToDate(DATE, -1)));
    }

    @Test
    public void startInDstEndOutOfDst() {
        final Date inDst = createDate(2006, 9, 1);
        final Date outDst = createDate(2006, 10, 1);

        assertEquals(31, DateTimeUtil.daysBetween(inDst, outDst));
    }

    @Test
    public void startOutDstEndInDst() {
        final Date inDst = createDate(2006, 9, 1);
        final Date outDst = createDate(2006, 10, 1);

        assertEquals(-31, DateTimeUtil.daysBetween(outDst, inDst));
    }

    @Test
    public void overLeapDayNoChangeInDst() {
        final Date beforeLeapDay = createDate(2004, 1, 27);
        final Date afterLeapDay = createDate(2004, 2, 1);

        assertEquals(3, DateTimeUtil.daysBetween(beforeLeapDay, afterLeapDay));
    }

    @Test
    public void overLeapDayAndOverDstChange() {
        final Date beforeLeapDayNonDst = createDate(2004, 1, 27);
        final Date afterLeapDayAndDst = createDate(2004, 3, 5);

        assertEquals(38, DateTimeUtil.daysBetween(beforeLeapDayNonDst,
                afterLeapDayAndDst));
    }
    
    @Test
    public void createDate() {
        final Date date = DateTimeUtil.createDateOn(10, 10, 2006);
        Calendar c = Calendar.getInstance();
        c.setTime(date);
        assertEquals(10, c.get(Calendar.DAY_OF_MONTH));
        assertEquals(Calendar.OCTOBER, c.get(Calendar.MONTH));
        assertEquals(2006, c.get(Calendar.YEAR));
    }
    
    @Test
    public void createDateWithDayOutOfMonthRange() {
        final Date date = DateTimeUtil.createDateOn(29, 2, 2006);
        Calendar c = Calendar.getInstance();
        c.setTime(date);
        assertEquals(1, c.get(Calendar.DAY_OF_MONTH));
        assertEquals(Calendar.MARCH, c.get(Calendar.MONTH));
        assertEquals(2006, c.get(Calendar.YEAR));
    }

    private Date addDaysToDate(final Date date, final int days) {
        Calendar c = Calendar.getInstance();
        c.setTime(date);
        c.add(Calendar.DAY_OF_YEAR, days);
        return c.getTime();
    }

    private Date createDate(final int year, final int month, final int day) {
        final Calendar c = Calendar.getInstance();
        c.set(Calendar.YEAR, year);
        c.set(Calendar.MONTH, month);
        c.set(Calendar.DAY_OF_MONTH, day);

        return c.getTime();
    }
}
{% endhighlight %}

## Review Existing Tests
We have the following Test classes in the session package:
* BookDaoTest
* LibraryTest
* PatronDaoTest

BookDaoTest and PatronDao test deal with CRUD operations on Book and Patron. They don't really care about the structure of the objects since they just read/write Book and Patron objects. We might need to make changes to the Book and Patron entities to keep these tests passing, but we'll do that as tests fail.

### Review Tests in LibraryDaoTest
Here's our first test to review:
{% highlight java %}
    @Test
    public void checkoutBook() {
        final Book b = createBook();
        final Patron p = createPatron();
        library.checkout(p.getId(), b.getId());

        final Book foundBook = library.findBookById(b.getId());
        final Patron foundPatron = library.findPatronById(p.getId());

        assertTrue(foundBook.isOnLoanTo(foundPatron));
        assertTrue(foundPatron.isBorrowing(foundBook));
    }
{% endhighlight %}

This test might work as is, however when we checkout a book, we do not provide a date. We could keep the interface the same and the library could assume the current date. Instead, we'll provide a date and then verify that the book was checked out on the date provided. Since we will be in a state of flux for some time, we'll add a new method and deprecate the old method. By the time we've finished this update, we'll remove all deprecated methods.
