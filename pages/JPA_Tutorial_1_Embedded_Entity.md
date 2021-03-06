---
title: JPA_Tutorial_1_Embedded_Entity
---
When we created Person we directly included address information into them. This is alright, but what if we want to use Address in another class? Let's introduce a new entity, Address, and make it **embedded**. This means its fields will end up as columns in the table of the entity that contains it.

First, we'll create Address:
{: #Address}
### Address.java
{% highlight java %}
package entity;

import javax.persistence.Embeddable;

@Embeddable
public class Address {
    private String streetAddress1;
    private String streetAddress2;
    private String city;
    private String state;
    private String zip;

    public Address() {
    }

    public Address(final String sa1, final String sa2, final String city,
            final String state, final String zip) {
        setStreetAddress1(sa1);
        setStreetAddress2(sa2);
        setCity(city);
        setState(state);
        setZip(zip);
    }

    public final String getCity() {
        return city;
    }

    public final void setCity(final String city) {
        this.city = city;
    }

    public final String getState() {
        return state;
    }

    public final void setState(final String state) {
        this.state = state;
    }

    public final String getStreetAddress1() {
        return streetAddress1;
    }

    public final void setStreetAddress1(final String streetAddress1) {
        this.streetAddress1 = streetAddress1;
    }

    public final String getStreetAddress2() {
        return streetAddress2;
    }

    public final void setStreetAddress2(final String streetAddress2) {
        this.streetAddress2 = streetAddress2;
    }

    public final String getZip() {
        return zip;
    }

    public final void setZip(final String zip) {
        this.zip = zip;
    }
}
{% endhighlight %}

Next, we need to update Person (doing so will cause our unit test class to no longer compile):
### Person.java
{% highlight java %}
package entity;

import javax.persistence.Embedded;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Person {
    @Id
    @GeneratedValue
    int id;
    private String firstName;
    private char middleInitial;
    private String lastName;

    @Embedded
    private Address address;

    public Person() {
    }

    public Person(final String fn, final char mi, final String ln,
            final Address address) {
        setFirstName(fn);
        setMiddleInitial(mi);
        setLastName(ln);
        setAddress(address);
    }

    public String getFirstName() {
        return firstName;
    }

    public void setFirstName(final String firstName) {
        this.firstName = firstName;
    }

    public int getId() {
        return id;
    }

    public void setId(final int id) {
        this.id = id;
    }

    public String getLastName() {
        return lastName;
    }

    public void setLastName(final String lastName) {
        this.lastName = lastName;
    }

    public char getMiddleInitial() {
        return middleInitial;
    }

    public void setMiddleInitial(final char middleInitial) {
        this.middleInitial = middleInitial;
    }

    public final Address getAddress() {
        return address;
    }

    public final void setAddress(final Address address) {
        this.address = address;
    }
}
{% endhighlight %}

Sure enough, if you review PersonTest.java, it no longer compiles. Before we go any further, let's update it to get it to compile and then verify that the unit tests still pass.

Replace the following two lines:
{% highlight java %}
    private final Person p1 = new Person("Brett", 'L', "Schuchert", "Street1",
            "Street2", "City", "State", "Zip");
    private final Person p2 = new Person("FirstName", 'K', "LastName",
            "Street1", "Street2", "City", "State", "Zip");
{% endhighlight %}

with the following four lines

{% highlight java %}
    private final Address a1 = new Address("A Rd.", "", "Dallas", "TX", "75001");
    private final Person p1 = new Person("Brett", 'L', "Schuchert", a1);

    private final Address a2 = new Address("B Rd.", "S2", "OkC", "OK", "73116");
    private final Person p2 = new Person("FirstName", 'K', "LastName", a2);
{% endhighlight %}

Rerun your tests (**Ctrl-F11**) and make sure everything is all green.

Next, we want to verify that the address we persist is in the database. Update the unit test method as follows:

### PersonTest#insertAndRetrieve
{% highlight java %}
    @SuppressWarnings("unchecked")
    @Test
    public void insertAndRetrieve() {
        em.getTransaction().begin();
        em.persist(p1);
        em.persist(p2);
        em.getTransaction().commit();

        final List<Person> list = em.createQuery("select p from Person p")
                .getResultList();

        assertEquals(2, list.size());
        for (Person current : list) {
            final String firstName = current.getFirstName();
            final String streetAddress1 = current.getAddress()
                    .getStreetAddress1();

            assertTrue(firstName.equals("Brett")
                    || firstName.equals("FirstName"));
            assertTrue(streetAddress1.equals("A Rd.")
                    || streetAddress1.equals("B Rd."));
        }
    }
{% endhighlight %}

Run your program and make sure it's all green.
