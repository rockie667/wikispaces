---
title: FitNesse.Tutorials.TableTables
---
{% include nav prev="FitNesse.Tutorials" next="FitNesse.Tutorials.AlternativeScriptTableSyntax" %}
{% include toc %}

## Background
This is a tutorial loosely based on [this writeup](AcceptanceTesting.FitNesse.TableTableExample). [That writeup](AcceptanceTesting.FitNesse.TableTableExample) describes using table table to implement test data setup to make determining expected results easier. You can read that for a slightly different take. That example was written after the fact and somewhat cleaned up. It also is not a tutorial; it is really a summary of what you'll be doing in this tutorial.

In this tutorial, you'll review the setup for a previous test and then build the test setup in a way that will much better relate to the domain. Unlike the [original table table example](AcceptanceTesting.FitNesse.TableTableExample), this one will seem a lot more like a plausible development effort.

## Getting Started
As with the other tutorials, you can continue from the work you've done on the [previous tutorial](FitNesse.Tutorials.ScenarioTables), or you can [use the source](FitNesse.Tutorials.WorkingFromGitHub) and start at the tag: FitNesse.Tutorials.TableTables.Start.

Up to this point, you have created programs using several different styles. However, all of these styles are very different from the underlying domain. This tutorial picks up from the [Scenario Tables Tutorial](FitNesse.Tutorials.ScenarioTables) and looks at one final way to create a program guide, or a series of programs.

## Creating Many Programs
You used the following table to populate the program schedule (this is a snippet):
{% highlight terminal %}
|Create Daily Program Named|D5_1|On Channel|5|Starting On|3/4/2008|at|20:00|Length|30|Episodes|7|
|Create Daily Program Named|D5_2|On Channel|5|Starting On|3/4/2008|at|20:30|Length|30|Episodes|7|
|Create Daily Program Named|D5_3|On Channel|5|Starting On|3/4/2008|at|21:00|Length|30|Episodes|7|
|Create Daily Program Named|D5_4|On Channel|5|Starting On|3/4/2008|at|21:30|Length|30|Episodes|7|
|Create Daily Program Named|D6_1|On Channel|6|Starting On|3/4/2008|at|20:00|Length|30|Episodes|7|
|Create Daily Program Named|D6_2|On Channel|6|Starting On|3/4/2008|at|20:30|Length|30|Episodes|7|
|Create Daily Program Named|D6_3|On Channel|6|Starting On|3/4/2008|at|21:00|Length|30|Episodes|7|
|Create Daily Program Named|D6_4|On Channel|6|Starting On|3/4/2008|at|21:30|Length|30|Episodes|7|
...
{% endhighlight %}

What does this table represent? What follows is a different visualization of this same data to make it easier to understand. In fact, it is just two days, but you can probably imagine extending this to support multiple days:
{% highlight terminal %}
|3/4/2008|20:00  |20:30  |21:00  |21:30  |
|5       |D5_1.E1|D5_2.E1|D5_3.E1|D5_4.E1|
|6       |D6_1.E1|D6_2.E1|D6_3.E1|D6_4.E1|
|3/5/2008|20:00  |20:30  |21:00  |21:30  |
|5       |D5_1.E1|D5_2.E1|D5_3.E1|D5_4.E1|
|6       |D6_1.E1|D6_2.E1|D6_3.E1|D6_4.E1|
{% endhighlight %}

Notice that this appears more like a program guide you might see with cable receiver or a DVR. This isn't perfect because there are artifacts from the original table in the forms of the names used to create the programs. Even so, this appears to be a bit closer to a program guide.

Next, consider the following simplifications:
* Programs only have one episode called "E1"
* Only consider one day
* Simplify the names. The name of the program/episode is just a sequence of the same characters, e.g. aaaabbbb, represents 2 programs, one called aaaa and the second called bbbb.
* Rather than specify each starting time, make the length of the name related to the program length, e.g., 4 letters is 1 hour, 1 letter is 15 minutes.

With these changes, read this program guide:

{% highlight terminal %}
|Table:Create One Day Program Guide|1:00|3/4/2008| 
|   |1   |2   |3   |4   |5   |6   |7   |8   |9   |10  |11  |12  |13  |14  |
|200|aaaa|BBcc|cccc|ccDD|DDee|efff|ffff|fffg|gggg|gggh|hhii|jklm|nopq|rstt|
|247|aaaa|BBBB|cccc|DDDD|eeee|FFFF|gggg|HHHH|iiii|JJJJ|kkkk|LLLL|mmmm|NNNN|
|302|aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|ssTT|uuVV|wwww|wwXX|XXXy|
|501|    |    |    |    |    |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|
|556|    |__aa|BBcc|DDee|FFgg|HHii|JJkk|LLmm|NNoo|PPqq|RRss|TTuu|VVxx|xxxx|
{% endhighlight %}

Why do this? When I was testing the logic of selecting the correct programs with multiple season passes and a variable number of recorders, I had trouble program the schedules using the script table. What I was doing in my head was visualizing the program guide used by the DVRs I've personally used. It then hit me that the visualization of the program guide was essential to my use of the DVR. My tests did not reflect that so it occurred to me to make my tests reflect that as close as I could.

There is one problem with this setup. On DVRs, the length of the program is not related to the length of its name. This is an artificial simplification to make the table representation look decent. Even so, in practice this representation made writing tests easier at the expense of a more complex fixture, and that's the right choice to make.

What you will do in the remainder of this tutorial is create a fixture to handle this new table type. Once you've done that, you'll recreate some of the tests from the previous tutorial using the table table for the setup.

## Creating the table
As with the previous tutorials, you'll create these tests under their own sub-hierarchy:
* Create a suite-page for these tests here: <http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.TableTableExamples>
* Just save the page as is (with the !contents ...)
* Change its type to a suite page (**Note**, as of 4/15/09, FitNesse has been updated to automatically set the page type to a suite if it ends in "Examples". If you build from source or you happen to have a recent release with this feature built, you might not need to set the type to suite.)
* Add a SetUp page. If the link exists on the [suite page](http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.TableTableExamples), you can click it (this feature was there, then removed and I added it back in again). If not, then go to this URL to create the setup page: <http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.TableTableExamples.SetUp>
* Set its contents to:

{% highlight terminal %}
!include <DigitalVideoRecorderExamples.SetUp

|Table:Create One Day Program Guide|1:00|3/4/2008|
|   |1   |2   |3   |4   |5   |6   |7   |8   |9   |10  |11  |12  |13  |14  |
|200|aaaa|BBcc|cccc|ccDD|DDee|efff|ffff|fffg|gggg|gggh|hhii|jklm|nopq|rstt|
|247|aaaa|BBBB|cccc|DDDD|eeee|FFFF|gggg|HHHH|iiii|JJJJ|kkkk|LLLL|mmmm|NNNN|
|302|aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|ssTT|uuVV|wwww|wwXX|XXXy|
|501|    |    |    |    |    |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|
|556|    |__aa|BBcc|DDee|FFgg|HHii|JJkk|LLmm|NNoo|PPqq|RRss|TTuu|VVxx|xxxx|

!|Scenario|dvrCanSimultaneouslyRecord|number|andWithThese|seasonPasses|shouldHaveTheFollowing|toDoList|
|givenDvrCanRecord|@number|
|whenICreateSeasonPasses|@seasonPasses|
|thenTheToDoListShouldContain|@toDoList|

|Script|Dvr Recording|
{% endhighlight %}

Notice that this borrows the Scenario table and script table from the [SetUp](http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.ScenarioTableExamples.SetUp) page in the [previous tutorial](FitNesse.Tutorials.ScenarioTables). In this tutorial, the only new table is a new Table Table to populate the program schedule. 

Next, we need a test that uses this <http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.TableTableExamples.SetUp>
* Create a new test page: <http://localhost:8080/FrontPage.DigitalVideoRecorderExamples.TableTableExamples.DvrThatCanRecordTwoProgramsAtaTimeExample>
* Set its contents to: 

{% highlight terminal %}
|Scenario|A Two Recorder Dvr With These Season Passes|seasonPasses|Should Have These Episodes In To Do List|toDoList|
|Dvr Can Simultaneously Record | 2 | And With These |@seasonPasses|Should Have The Following|@toDoList|

|A Two Recorder Dvr With These Season Passes Should Have These Episodes In To Do List|
|seasonPasses                                |toDoList                               |
|cccccccc:200,FF:302                         |cccccccc:E:1-1,FF:E:1-1                |
{% endhighlight %}

* You might need to set this page to a test page. As of 4/15/09, the FitNesse source will automatically set the test page type for pages that begin with or end with the word "Example" (in addition to the word "Test"). However, you might not have the latest release.
* Finally, you have a complete test with SetUp and TearDown code. Run it and notice that the test fails. It cannot find the class "Create One Day Program Guide", which is required by the Table Table.
### Create the Fixture
* Create version 1 of the fixture:

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.util.Collections;
import java.util.List;

public class CreateOneDayProgramGuide {
   public CreateOneDayProgramGuide(String startTime, String date) {
   }

   public List<?> doTable(List<List<String>> table) {
      return Collections.emptyList();
   }
}
{% endhighlight %}

While this Fixture does not do anything yet, it is a minimal example that will get the test to run, finding all of the Fixtures. The test is still failing.

The minimal requirement for the fixture is a doTable method as shown above. Since the table takes in parameters to its constructor, this fixture also needs a matching constructor. Now that you have the basic infrastructure in place, it's time to experiment just a little bit to see just what FitNesse really passes in to the doTable method.

* Update the doTable method:
{% highlight java %}
   public List<?> doTable(List<List<String>> table) {
      for (List<String> row : table) {
         System.out.print("|");
         for (String cell : row)
            System.out.printf("%s|", cell);
         System.out.println();
      }
      return Collections.emptyList();
   }
{% endhighlight %}

* Execute this test and review the output (click on the yellow triangle with an ! in it):

{% highlight terminal %}
|1|2|3|4|5|6|7|8|9|10|11|12|13|14|
|200|aaaa|BBcc|cccc|ccDD|DDee|efff|ffff|fffg|gggg|gggh|hhii|jklm|nopq|rstt|
|247|aaaa|BBBB|cccc|DDDD|eeee|FFFF|gggg|HHHH|iiii|JJJJ|kkkk|LLLL|mmmm|NNNN|
|302|aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|ssTT|uuVV|wwww|wwXX|XXXy|
|501|||aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|
|556|__aa|BBcc|DDee|FFgg|HHii|JJkk|LLmm|NNoo|PPqq|RRss|TTuu|VVxx|xxxx|
{% endhighlight %}

Does this look familiar? This gives a hint at just what a table-table does. FitNesse simply passes in the entire table (minus the first row) in the form of a list of a list of strings:
* The outside list represents the collection of rows. The first entry is actually the second row of the actual table, FitNesse does not pass in the row naming the fixture. (We're going to ignore that row altogether in this tutorial, it's there for documentation).
* The inside list represents the individual cells within a given row. In our case, the first cell is the channel. The remaining cells represent an hour of programming.

With that basic understanding, now it is time to process an individual row. This fixture (and in general, table-table fixtures) can be complex enough to warrant unit test code. Why is that? You are trying to make a table that is easy for a non-programmer to be able to use effectively; something that is closer to the problem domain. Because the table is closer to the domain and further away from the implementation, it will require some amount of coding.
## Switch to Unit Testing
Our fixture needs to be able process a series of rows, each of which represent a channel of programming. That's where we'll start with unit testing.
### Create the First Test
* This first test simply puts most of the basic API in place:

{% highlight java %}
package com.om.example.dvr.fixtures;

import static org.junit.Assert.assertEquals;

import java.util.List;

import org.junit.Test;

import com.om.example.dvr.domain.Program;

public class ProgramGuideRowParserTest {
   @Test
   public void emptyRowGeneratesNoPrograms() {
      ProgramGuideRowParser parser = new ProgramGuideRowParser();
      List<Program> result = parser.parse("");
      assertEquals(0, result.size());
   }
}
{% endhighlight %}

**Purpose**: The purpose of this test is to being defining the API by which row parsing will happen.
* Here's the minimal code to make this test pass:

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.util.LinkedList;
import java.util.List;

import com.om.example.dvr.domain.Program;

public class ProgramGuideRowParser {

   public List<Program> parse(String string) {
      return new LinkedList<Program>();
   }
}
{% endhighlight %}

* Run the test, make sure it passes.
* Next, add another test with one program and notice that this requires several changes:
#### Update: ProgramGuideRowParserTest, Add new test

{% highlight java %}
package com.om.example.dvr.fixtures;

import static org.junit.Assert.assertEquals;

import java.text.ParseException;
import java.util.Date;
import java.util.List;

import org.junit.Before;
import org.junit.Test;

import com.om.example.dvr.domain.Program;
import com.om.example.dvr.domain.TimeSlot;
import com.om.example.util.DateUtil;

public class ProgramGuideRowParserTest {
   private ProgramGuideRowParser parser;

   @Before
   public void init() throws ParseException {
      parser = new ProgramGuideRowParser(DateUtil.instance()
            .buildDate("4/8/2008", "1:00"));
   }

   @Test
   public void emptyRowGeneratesNoPrograms() {
      List<Program> result = parser.parse("");
      assertEquals(0, result.size());
   }

   @Test
   public void oneOneHourProgram() throws ParseException {
      parser.setChannel(204);
      List<Program> result = parser.parse("|aaaa|");

      assertEquals(1, result.size());

      Program expected = buildProgram("4/8/2008", "1:00", "aaaa", 204, 60);
      assertEquals(expected, result.get(0));
   }

   private Program buildProgram(String date, String time, String name, int channel,
         int duration) throws ParseException {
      Date expectedStartDateTime = DateUtil.instance().buildDate(date, time);
      return new Program(name, "E1", new TimeSlot(channel, expectedStartDateTime,
            duration));
   }
}
{% endhighlight %}

#### Update ProgramGuidRowPaser: Add constructor and method

{% highlight java %}
   public ProgramGuideRowParser(Date buildDate) {
   }

   public void setChannel(int channel) {
   }
{% endhighlight %}

* Run your tests, they fail.
* Add missing equals() methods:

**Update: TimeSlot**:

{% highlight java %}
   @Override
   public boolean equals(Object other) {
      if (!(other instanceof TimeSlot))
         return false;

      TimeSlot rhs = getClass().cast(other);
      return channel == rhs.channel && durationInMinutes == rhs.durationInMinutes
            && startDateTime.equals(rhs.startDateTime);
   }
{% endhighlight %}

#### Update: Program

{% highlight java %}
   @Override
   public boolean equals(Object other) {
      if (!(other instanceof Program))
         return false;

      Program rhs = getClass().cast(other);
      return programName.equals(rhs.programName) && episodeName.equals(rhs.episodeName)
            && timeSlot.equals(rhs.timeSlot);
   }
{% endhighlight %}
* Finally, update the production code to get this test passing:

{% highlight java %}
public class ProgramGuideRowParser {
   private int channel;
   private final Date buildDate;

   public ProgramGuideRowParser(Date buildDate) {
      this.buildDate = buildDate;
   }

   public void setChannel(int channel) {
      this.channel = channel;
   }

   public List<Program> parse(String programsInCells) {
      LinkedList<Program> result = new LinkedList<Program>();

      if (programsInCells.length() > 0) {
         TimeSlot timeSlot = new TimeSlot(channel, buildDate, 60);
         result.add(new Program("aaaa", "E1", timeSlot));
      }

      return result;
   }
}
{% endhighlight %}
* Run your tests, they should pass.
**Note**: Did you notice that you just added equals() methods to Program and TimeSlot without adding unit tests to those same classes? Is this a problem? These methods are complex enough that there is certainly some risk in adding them. Also, in Java it is standard practice to write hashCode() when writing equals() just in case the object is used as a key in a Map. However, the code does not sore these objects as keys in Maps, so writing a hashCode() method, while conventional, is not really necessary.

These methods were written in response to a test, something more than a unit test, but a test none the less. Whether to add tests for the equals() method beyond what we've already written is not a clear yes or no decision, so I'll leave that to the reader since this is more about working with FitNesse than unit testing (in the book version of this tutorial, however, I'll probably take the other approach).

### Next Test: Getting Program Length Correct
* Add this test:

{% highlight java %}
   @Test
   public void oneThirtyMinuteProgram() throws ParseException {
      parser.setChannel(204);
      List<Program> result = parser.parse("|aa|");

      assertEquals(1, result.size());

      Program expected = buildProgram("4/8/2008", "1:00", "aa", 204, 30);
      assertEquals(expected, result.get(0));
   }
{% endhighlight %}

* Run it and verify that it does not pass. 
* Next, update the production code in ProgramGuideRowParser:

{% highlight java %}
   public List<Program> parse(String programsInCells) {
      List<Program> result = new LinkedList<Program>();

      if (programsInCells.length() > 0) {
         String[] cells = programsInCells.substring(1).split("\\|");

         String currentName = cells[0];
         TimeSlot timeSlot = new TimeSlot(channel, buildDate, currentName.length() * 15);
         result.add(new Program(currentName, "E1", timeSlot));
      }

      return result;
   }
{% endhighlight %}

* Run your tests, make sure they pass.
**Note**: This is a somewhat refactored method. It will get longer and shorter as you work through this parsing exercise.

### Next Test: Handle two 30 minute programs
* Add a new test (and update the @Before method):

{% highlight java %}
   @Before
   public void init() throws ParseException {
      parser = new ProgramGuideRowParser(DateUtil.instance()
            .buildDate("4/8/2008", "1:00"));
      parser.setChannel(204);
   }

   // ...

   @Test
   public void twoThirtyMinuteProgramsInSameCell() throws ParseException {
      List<Program> result = parser.parse("|aabb|");
      
      assertEquals(2, result.size());
      
      Program expected0 = buildProgram("4/8/2008", "1:00", "aa", 204, 30);
      assertEquals(expected0, result.get(0));
      
      Program expected1 = buildProgram("4/8/2008", "1:30", "bb", 204, 30);
      assertEquals(expected1, result.get(1));
   }
{% endhighlight %}

**Note**: You can remove the parser.setChannel(204) from the other two @Test methods since that duplicated method has be pushed up into the @Before method.
* Run your tests, the new one will.
* Now make several updates to make this next test pass (and notice that the code is getting unruly):

#### Update: ProgramGuideRowParser.java

{% highlight java %}
   public List<Program> parse(String programsInCells) {
      List<Program> result = new LinkedList<Program>();

      if (programsInCells.length() > 0) {
         Date nextDate = buildDate;

         List<String> programNames = split(programsInCells);

         for (String currentName : programNames) {
            int length = currentName.length() * 15;

            TimeSlot timeSlot = new TimeSlot(channel, nextDate, length);
            result.add(new Program(currentName, "E1", timeSlot));
            nextDate = calculateNextDate(nextDate, length);
         }
      }
      return result;
   }

   private List<String> split(String programsInCells) {
      String[] cells = programsInCells.substring(1).split("\\|");

      List<String> result = new LinkedList<String>();

      int lastStartIndex = 0;
      int lastChar = cells[0].charAt(0);
      for (int i = 1; i < cells[0].length(); ++i) {
         if (cells[0].charAt(i) != lastChar) {
            String lastProgramName = cells[0].substring(lastStartIndex, i);
            result.add(lastProgramName);
            lastStartIndex = i;
            lastChar = cells[0].charAt(i);
            continue;
         }
         if (i == cells[0].length() - 1) {
            String lastProgramName = cells[0].substring(lastStartIndex);
            result.add(lastProgramName);
         }
      }

      return result;
   }

   private Date calculateNextDate(Date fromDate, int lengthInMinutes) {
      return DateUtil.instance().addMinutesTo(fromDate, lengthInMinutes);
   }
{% endhighlight %}

#### Update: DateUtil.java

{% highlight java %}
   public Date addMinutesTo(Date fromDate, int minutes) {
      Calendar calendar = Calendar.getInstance();
      calendar.setTime(fromDate);
      calendar.add(Calendar.MINUTE, minutes);
      return calendar.getTime();
   }
{% endhighlight %}
* Run your tests, make sure they pass.

### Next Test: Ignore _ in name
* FitNesse removes extra spaces on either side of the cell. To represent "no program", the example uses _. Here's a test that verifies the production code handles _'s correctly:

{% highlight java %}
   @Test
   public void underscoredIgnored() throws ParseException {
      List<Program> result = parser.parse("|__aa|");
      
      assertEquals(1, result.size());
      
      Program expected = buildProgram("4/8/2008", "1:30", "aa", 204, 30);
      assertEquals(expected, result.get(0));
   }
{% endhighlight %}

* After adding this test, verify that it fails.
* Now, update the parser to handle this condition (note, this is after several rounds of refactoring, with much refactoring still left):

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.util.Date;
import java.util.LinkedList;
import java.util.List;

import com.om.example.dvr.domain.Program;
import com.om.example.dvr.domain.TimeSlot;
import com.om.example.util.DateUtil;

public class ProgramGuideRowParser {

   private final Date buildDate;
   private int channel;

   public ProgramGuideRowParser(Date buildDate) {
      this.buildDate = buildDate;
   }

   public List<Program> parse(String programsInCells) {
      List<Program> result = new LinkedList<Program>();

      String programs = programsInCells.replaceAll("\\|", "");

      for (int index = startIndexOfNextProgram(0, programs); index < programs.length();) {
         Date nextDate = calculateNextDate(buildDate, index * 15);

         String nextProgram = getNextProgram(index, programs);
         if (nextProgram != null) {
            int length = nextProgram.length();
            index += length;

            result.add(buildProgram(nextDate, length * 15, nextProgram));
         }
      }

      return result;
   }

   private Program buildProgram(Date date, int duration, String name) {
      TimeSlot timeSlot = new TimeSlot(channel, date, duration);
      return new Program(name, "E1", timeSlot);
   }

   public void setChannel(int channel) {
      this.channel = channel;
   }

   private int startIndexOfNextProgram(int index, String string) {
      while (index < string.length() && !Character.isLetter(string.charAt(index)))
         ++index;
      return index;
   }

   private String getNextProgram(int index, String string) {
      int lastIndex = index;

      while (lastIndex < string.length()
            && string.charAt(lastIndex) == string.charAt(index))
         ++lastIndex;

      return string.substring(index, lastIndex);
   }

   private Date calculateNextDate(Date fromDate, int lengthInMinutes) {
      return DateUtil.instance().addMinutesTo(fromDate, lengthInMinutes);
   }
}
{% endhighlight %}

* Run your tests, make sure the pass.

### Next Test: A cell with all spaces handled correctly

* FitNesse will take an empty cell and pass in "", so here's a test to make sure the production code works: 

{% highlight java %}
   @Test
   public void emptyCellsHandeledCorrectly() throws ParseException {
      List<Program> result = parser.parse("||__aa|");
      
      assertEquals(1, result.size());
      
      Program expected = buildProgram("4/8/2008", "2:30", "aa", 204, 30);
      assertEquals(expected, result.get(0));
   }
{% endhighlight %}

* Run your tests, verify this new one fails.
* Update your parser to get this test to pass:

{% highlight java %}
   public List<Program> parse(String programsInCells) {
      List<Program> result = new LinkedList<Program>();

      String programs = buildSingleStringFromCells(programsInCells);

      // snip - unchanged
   }

   private String buildSingleStringFromCells(String programsInCells) {
      String programs = programsInCells.replaceAll("\\|\\|", "|    |");
      programs = programs.replaceAll("\\|", "");
      return programs;
   }
{% endhighlight %}

* Run your tests, make sure they pass.

### Final Test: One Big Row

This algorithm is either close to complete or complete. Here's a final test that will bring everything together:

{% highlight java %}
   @Test
   public void verifyComplexParse() throws ParseException {
      List<Program> result = parser.parse("||__aa|BBcc||FFgg|__  |");
      assertEquals(5, result.size());

      Program expectedLastProgram = buildProgram("4/8/2008", "5:30", "gg", 204, 30);
      assertEquals(expectedLastProgram, result.get(4));
   }
{% endhighlight %}

* Run your tests, make sure this new test fails (the underlying code is close but not quite there).
* Here's the update to the parse method (post-refactoring):

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.util.Date;
import java.util.LinkedList;
import java.util.List;

import com.om.example.dvr.domain.Program;
import com.om.example.dvr.domain.TimeSlot;
import com.om.example.util.DateUtil;

public class ProgramGuideRowParser {

   private static final int MINS_PER_CHAR = 15;
   private final Date buildDate;
   private int channel;

   public ProgramGuideRowParser(Date buildDate) {
      this.buildDate = buildDate;
   }

   public List<Program> parse(String programsInCells) {
      List<Program> result = new LinkedList<Program>();

      String programs = buildSingleStringFromCells(programsInCells);

      int index = 0;
      while ((index = startIndexOfNextProgram(index, programs)) < programs.length()) {
         String nextProgramName = calculateNextProgramName(index, programs);
         index += addNextProgram(nextProgramName, index, result);
      }

      return result;
   }

   public void setChannel(int channel) {
      this.channel = channel;
   }

   private String buildSingleStringFromCells(String programsInCells) {
      String programs = programsInCells.replaceAll("\\|\\|", "|    |");
      programs = programs.replaceAll("\\|", "");
      return programs;
   }

   private Program buildProgram(int index, int duration, String name) {
      Date nextDate = calculateNextDate(buildDate, index * MINS_PER_CHAR);
      TimeSlot timeSlot = new TimeSlot(channel, nextDate, duration);
      return new Program(name, "E1", timeSlot);
   }

   private int addNextProgram(String programName, int index, List<Program> result) {
      if (programName != null) {
         int length = programName.length();
         result.add(buildProgram(index, length * MINS_PER_CHAR, programName));
         return length;
      }
      return 0;
   }

   private int startIndexOfNextProgram(int index, String string) {
      while (index < string.length() && !Character.isLetter(string.charAt(index)))
         ++index;
      return index;
   }

   private String calculateNextProgramName(int index, String string) {
      int lastIndex = index;

      while (lastIndex < string.length() && charactesSameAt(string, index, lastIndex))
         ++lastIndex;

      return string.substring(index, lastIndex);
   }

   private boolean charactesSameAt(String string, int startIndex, int endIndex) {
      return string.charAt(endIndex) == string.charAt(startIndex);
   }

   private Date calculateNextDate(Date fromDate, int lengthInMinutes) {
      return DateUtil.instance().addMinutesTo(fromDate, lengthInMinutes);
   }
}
{% endhighlight %}

* Make this change, verify that your tests pass.
* Make sure all of your unit tests still pass.
* Switch back to your browser and verify that all acceptance tests still pass (other than the one failing test).

## Bringing it all together
Now that you can parse a single row, there are a few things left before your table-table fixture will be ready:
* Parse a row that contains both a channel and all of the programs.
* Take all of the programs from all of the rows and add them to the program schedule.

This will require several more steps, so let's get started.

### Refactor: The Name is wrong
The name of the parser class is wrong, it only parses the programs not the whole row (it does not handle the channel).
* Rename the class ProgramGuideRowParser --> ProgramGuideProgramCellsParser.
* Run your tests, make sure they still pass
* Rename the test class ProgramGuidRowParserTest --> ProgramGuideProgramCellsParserTest
* Run your tests, make sure they still pass.

### Create the Real ProgramGuideRowParser
* Create a new test class, ProgramGuideRowParserTest:

{% highlight java %}
package com.om.example.dvr.fixtures;

import static org.junit.Assert.assertEquals;

import java.text.ParseException;
import java.util.List;

import org.junit.Test;

import com.om.example.dvr.domain.Program;

public class ProgramGuideRowParserTest {
   @Test
   public void verifyChannelSetCorrectly() throws ParseException {
      ProgramGuideRowParser parser = new ProgramGuideRowParser("4/5/2008", "9:00");
      List<Program> programs = parser.parse("|123|aaaa|");

      assertEquals(1, programs.size());
      Program expected = ProgramBuilderUtil.buildProgram("4/5/2008", "9:00", "aaaa", 123,
            60);
      assertEquals(expected, programs.get(0));
   }
}
{% endhighlight %}

* This requires two new classes:

#### Create: ProgramBuilderUtil.java

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.text.ParseException;
import java.util.Date;

import com.om.example.dvr.domain.Program;
import com.om.example.dvr.domain.TimeSlot;
import com.om.example.util.DateUtil;

public class ProgramBuilderUtil {
   public static Program buildProgram(String date, String time, String name, int channel,
         int duration) throws ParseException {
      Date expectedStartDateTime = DateUtil.instance().buildDate(date, time);
      return new Program(name, "E1", new TimeSlot(channel, expectedStartDateTime,
            duration));
   }
}
{% endhighlight %}

#### Create: ProgramGuideRowParser

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.text.ParseException;
import java.util.Date;
import java.util.List;

import com.om.example.dvr.domain.Program;
import com.om.example.util.DateUtil;

public class ProgramGuideRowParser {
   ProgramGuideProgramCellsParser parser;

   public ProgramGuideRowParser(String date, String startTime) throws ParseException {
      Date startDateTime = DateUtil.instance().buildDate(date, startTime);
      parser = new ProgramGuideProgramCellsParser(startDateTime);
   }

   public List<Program> parse(String row) {
      int channel = getChannelFrom(row);
      parser.setChannel(channel);
      return parser.parse(justProgramCellsFrom(row));
   }

   private String justProgramCellsFrom(String row) {
      return row.replaceAll("^\\|\\d+\\|", "");
   }

   private int getChannelFrom(String row) {
      String justTime = row.substring(1).replaceAll("\\|.*", "");
      return Integer.parseInt(justTime);
   }
}
{% endhighlight %}

* Also, the new class ProgramBuilderUtil was extracted from the previous class:

#### Update: ProgramGuideProgramCellsParserTest.java

{% highlight java %}
   private Program buildProgram(String date, String time, String name, int channel,
         int duration) throws ParseException {
      return ProgramBuilderUtil.buildProgram(date, time, name, channel, duration);
   }
{% endhighlight %}

* Run all of your unit tests, make sure everything passes.

### Finally, upate the Table Fixture
All the pieces are in place, now it's just a matter of creating the programs:
* Update the CreateOneDayProgramGuide.java**

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.text.ParseException;
import java.util.Collections;
import java.util.List;

import com.om.example.dvr.domain.Program;

public class CreateOneDayProgramGuide {
   private ProgramGuideRowParser parser;

   public CreateOneDayProgramGuide(String startTime, String date) throws ParseException {
      parser = new ProgramGuideRowParser(date, startTime);
   }

   public List<?> doTable(List<List<String>> table) {
      table.remove(0);

      for (List<String> row : table)
         handleOneRow(row);

      return Collections.emptyList();
   }

   private void handleOneRow(List<String> row) {
      String string = rowAsString(row);

      List<Program> programs = parser.parse(string);

      for (Program program : programs) 
         AddProgramsToSchedule.getSchedule().addProgram(program);
   }

   private String rowAsString(List<String> row) {
      StringBuffer buffer = new StringBuffer();

      buffer.append("|");
      for (String current : row)
         buffer.append(String.format("%s|", current));

      return buffer.toString();
   }
}
{% endhighlight %}

* You'll need to add one method to Schedule:

{% highlight java %}
   public void addProgram(Program program) {
      if (conflictsWithOtherTimeSlots(program.timeSlot))
         throw new ConflictingProgramException();

      scheduledPrograms.add(program);
   }
{% endhighlight %}
* Make sure all of your unit tests pass.
* Make sure all of your acceptance tests pass.

## Final Cleanup
I made some false starts. For example, rather than having the ProgramGuideRowParser take in a String, it could easily have taken in a List<String> and did all of its work based on that. Also, the DvrRecording fixture requires a range of episodes, but the way this program fixture works, it only creates one episode per program. I'm sure you could review the fixtures and also the unit tests and production code and find several more places to refactor. Before you leave this tutorial, let's finish up with some basic refactoring of the Fixtures.

### Allow for a Single Episode
After a little experimentation with the DvrRecording fixture, I discovered a simple change that allows a single value rather than a range. I tried a few values and ran the tests after each time.
* Make the following update to DvrRecording (note, only the return statement in each of these methods changed):

{% highlight java %}
   private int extractLowerRangeFrom(String episodeSet) {
      // snip

      return Integer.MAX_VALUE;
   }

   private int extractUpperRangeFrom(String episodeSet) {
      // snip

      return 1;
	}
{% endhighlight %}

* Run all of your unit tests, make sure they still pass.
* Run all of your acceptance tests, make sure the still pass.
* Update your acceptance test:

{% highlight terminal %}
|A Two Recorder Dvr With These Season Passes Should Have These Episodes In To Do List|
|seasonPasses                                |toDoList                               |
|cccccccc:200,FF:302                         |cccccccc:E:1,FF:E:1                    |
{% endhighlight %}

* Run your acceptance test, make sure they all pass. Close, but not quite.
* Make one more change to the DvrRecording fixture:

{% highlight java %}
   private int extractUpperRangeFrom(String episodeSet) {
      String[] values = episodeSet.split(":");
      if (values.length > 2 && values[2].indexOf('-') > 0) {
         String highRange = episodeSet.split(":")[2].split("-")[1];
         return Integer.parseInt(highRange);
      }
      return 1;
   }
{% endhighlight %}

* Run all of your unit tests and acceptance tests, they should all pass.

### Change ProgramGuideRowParser to take a List<String> instead of a String
* Update ProgramGuideRowParserTest:

{% highlight java %}
package com.om.example.dvr.fixtures;

import static org.junit.Assert.assertEquals;

import java.text.ParseException;
import java.util.LinkedList;
import java.util.List;

import org.junit.Test;

import com.om.example.dvr.domain.Program;

public class ProgramGuideRowParserTest {
   @Test
   public void verifyChannelSetCorrectly() throws ParseException {
      ProgramGuideRowParser parser = new ProgramGuideRowParser("4/5/2008", "9:00");
      List<Program> programs = parser.parse(buildList("123", "aaaa"));

      assertEquals(1, programs.size());
      Program expected = ProgramBuilderUtil.buildProgram("4/5/2008", "9:00", "aaaa", 123,
            60);
      assertEquals(expected, programs.get(0));
   }

   private List<String> buildList(String... strings) {
      List<String> result = new LinkedList<String>();

      for (String current : strings)
         result.add(current);

      return result;
   }
}
{% endhighlight %}

* Update ProgramGuidRowParser:

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.text.ParseException;
import java.util.Date;
import java.util.List;

import com.om.example.dvr.domain.Program;
import com.om.example.util.DateUtil;

public class ProgramGuideRowParser {
   ProgramGuideProgramCellsParser parser;

   public ProgramGuideRowParser(String date, String startTime) throws ParseException {
      Date startDateTime = DateUtil.instance().buildDate(date, startTime);
      parser = new ProgramGuideProgramCellsParser(startDateTime);
   }

   public List<Program> parse(List<String> cells) {
      int channel = Integer.parseInt(cells.get(0));
      cells.remove(0);
      parser.setChannel(channel);
      return parser.parse(rowAsString(cells));
   }

   private String rowAsString(List<String> row) {
      StringBuffer buffer = new StringBuffer();

      buffer.append("|");
      for (String current : row)
         buffer.append(String.format("%s|", current));

      return buffer.toString();
   }
}
{% endhighlight %}

* Update CreateOneDayProgramGuide:

{% highlight java %}
package com.om.example.dvr.fixtures;

import java.text.ParseException;
import java.util.Collections;
import java.util.List;

import com.om.example.dvr.domain.Program;

public class CreateOneDayProgramGuide {
   private ProgramGuideRowParser parser;

   public CreateOneDayProgramGuide(String startTime, String date) throws ParseException {
      parser = new ProgramGuideRowParser(date, startTime);
   }

   public List<?> doTable(List<List<String>> table) {
      table.remove(0);

      for (List<String> row : table)
         handleOneRow(row);

      return Collections.emptyList();
   }

   private void handleOneRow(List<String> row) {
      List<Program> programs = parser.parse(row);

      for (Program program : programs)
         AddProgramsToSchedule.getSchedule().addProgram(program);
   }
}
{% endhighlight %}

* Run your unit tests, verify they still pass.
* Run your acceptance tests, make sure they still pass.

## Conclusion and Summary
Congratulations, you have finished this tutorial.

This tutorial demonstrated that you can create tables reflecting a more natural or fluent style and then write more complex fixture code to support that style.

Once you created the table, you started creating a parser using TDD to build the parser from the ground up. While this tutorial did not show you all of the intermediate steps, it certainly did cover many of the major moving parts. Once you built most of the infrastructure, you tied it into the fixture and got the test passing.

Table tables are a great way to pass anything in that you'd like. You can also pass back a completely different table. That is not demonstrated here, this tutorial simply passes back an empty list so FitNesse disregards the return result, but it certainly is possible to pass back something different from the input table.

Notice that this tutorial was different from the previous tutorials in that you spent more time working on fixture-based code. Indeed, there was very little new code added to the production side. This is consistent with table fixtures, most of the work is in mapping the table representation into some kind of logical message. When you use table fixtures, you are making it easier for acceptance test writers to write better looking tests. You are taking the burden of the responsibility to make things work.

Assuming you've worked through the previous tutorials, you have now used all of the basic table types in Slim. Congratulations!

{% include nav prev="FitNesse.Tutorials" next="FitNesse.Tutorials.AlternativeScriptTableSyntax" %}
 
