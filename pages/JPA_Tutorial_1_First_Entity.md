---
title: JPA_Tutorial_1_First_Entity
---
For this example we'll use a "top-down" approach. This means we'll create a Plain Old Java Object (POJO) with some annotations to indicate how we want JPA to persist it. We're letting the EntityManager take care of creating the tables in the database for us.

### Create a Simple Class

The following class contains everything you need to begin persisting it to a database:
{: #Person}
### Person.java
{% highlight java %}
package entity;

import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;

@Entity
public class Person {
    @Id
    @GeneratedValue
    private int id;
    private String firstName;
    private char middleInitial;
    private String lastName;
    private String streetAddress1;
    private String streetAddress2;
    private String city;
    private String state;
    private String zip;

    public Person() {
    }

    public Person(final String fn, final char mi, final String ln,
            final String sa1, final String sa2, final String city,
            final String state, final String zip) {
        setFirstName(fn);
        setMiddleInitial(mi);
        setLastName(ln);
        setStreetAddress1(sa1);
        setStreetAddress2(sa2);
        setCity(city);
        setState(state);
        setZip(zip);
    }

    public String getCity() {
        return city;
    }

    public void setCity(final String city) {
        this.city = city;
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

    public String getState() {
        return state;
    }

    public void setState(final String state) {
        this.state = state;
    }

    public String getStreetAddress1() {
        return streetAddress1;
    }

    public void setStreetAddress1(final String streetAddress1) {
        this.streetAddress1 = streetAddress1;
    }

    public String getStreetAddress2() {
        return streetAddress2;
    }

    public void setStreetAddress2(final String streetAddress2) {
        this.streetAddress2 = streetAddress2;
    }

    public String getZip() {
        return zip;
    }

    public void setZip(final String zip) {
        this.zip = zip;
    }
}
{% endhighlight %}
### Update persistence.xml
Note, for our configuration this step is optional.

If you use libraries provided exclusively by JBoss and Company, then you do not need to update your persistence.xml. If you are using another vendor or you want to make sure that your solution will work regardless of your persistence provider, add the following line to your persistence.xml:
{% highlight xml %}
        <class>entity.Person</class>
{% endhighlight %}

Your updated persistence.xml is now:
{% highlight xml %}
<persistence>
    <persistence-unit name="examplePersistenceUnit" 
                      transaction-type="RESOURCE_LOCAL">
        <class>entity.Person</class>
        <properties>
            <property name="hibernate.show_sql" value="false" />
            <property name="hibernate.format_sql" value="false" />

            <property name="hibernate.connection.driver_class" 
                      value="org.hsqldb.jdbcDriver" />
            <property name="hibernate.connection.url" 
                      value="jdbc:hsqldb:mem:mem:aname" />
            <property name="hibernate.connection.username" value="sa" />

            <property name="hibernate.dialect" 
                      value="org.hibernate.dialect.HSQLDialect" />
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
{% endhighlight %}

### Inserting and Querying
Now we need to update our unit test class, Person.java. We will have it insert two people, query and verify that the people we created are in the database:
### PersonTest.java
{% highlight java %}
package entity;

import static org.junit.Assert.assertEquals;
import static org.junit.Assert.assertTrue;

import java.util.List;

import javax.persistence.EntityManager;
import javax.persistence.EntityManagerFactory;
import javax.persistence.Persistence;

import org.apache.log4j.BasicConfigurator;
import org.apache.log4j.Level;
import org.apache.log4j.Logger;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

public class PersonTest {
    private final Person p1 = new Person("Brett", 'L', "Schuchert", "Street1",
            "Street2", "City", "State", "Zip");
    private final Person p2 = new Person("FirstName", 'K', "LastName",
            "Street1", "Street2", "City", "State", "Zip");

    private EntityManagerFactory emf;
    private EntityManager em;

    @Before
    public void initEmfAndEm() {
        BasicConfigurator.configure();
        Logger.getLogger("org").setLevel(Level.ERROR);

        emf = Persistence.createEntityManagerFactory("examplePersistenceUnit");
        em = emf.createEntityManager();
    }

    @After
    public void cleanup() {
        em.close();
    }

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
            assertTrue(firstName.equals("Brett")
                    || firstName.equals("FirstName"));
        }
    }
}
{% endhighlight %}
Re-run this test (the short-cut for this is **Ctrl-Fll**). Verify that everything is green.
