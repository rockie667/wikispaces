---
title: FitNesse.Tutorials.AlternativeScriptTableSyntax
---
{% include nav prev="FitNesse.Tutorials" %}
{% include toc %}

## Introduction
Sometime in the beginning of 2010, Bob Martin started working on an alternative way to write script tables to provide something that looks more like a style used in tools like [cucumber](http://cukes.info/). While still somewhat preliminary, this tutorial gives you a brief introduction to using this alternative style. Before getting into details, there are a few things I'll note:
* Ultimately what gets executed in the fixture is the same thing that gets executed using "regular" scripts
* All processing is done by FitNesse before sending test information into Slim.
* There are two parts:
  * placeholder syntax for scenario tables
  * a new way to represent script tables

## Not Starting from Scratch
I'm bolting this tutorial on more than a year after writing the other tutorials. So I'm starting with the last checkin into github and creating all new tables for this example. I might eventually integrate this into the tutorial suite at some point in the future, but for now if you want to get started, [follow the instructions here](FitNesse.Tutorials.WorkingFromGitHub). When switching to tags, use the following tag: **FitNesse.FitNesse.Tutorials.TableTables.Finished**.

## An Example Revisited
As mentioned, this tutorial stands alone and is bolted on. As such, I'm going to provide a stand-alone page to get you started assuming you have no existing FitNesse pages already. What I realized picking this up a year later is I neglected to include the FitNesseRoot in git. I'll fix that on a rewrite.

Here is a complete example using existing fixtures with no change to existing code. What you can take from this is all of this processing is done in FitNesse before test information is ever set from FitNesse to Slim.
{% highlight terminal %}
!define TEST_SYSTEM {slim}
!path fitnesse.jar
!path /Users/schuchert/src/new_fitnesse_table_example/fitnesse-tutorials/DVR/bin

|import                     |
|com.om.example.dvr.fixtures|

|Table:Create One Day Program Guide|1:00|3/4/2008                                                        |
|                                  |1   |2   |3   |4   |5   |6   |7   |8   |9   |10  |11  |12  |13  |14  |
|200                               |aaaa|BBcc|cccc|ccDD|DDee|efff|ffff|fffg|gggg|gggh|hhii|jklm|nopq|rstt|
|247                               |aaaa|BBBB|cccc|DDDD|eeee|FFFF|gggg|HHHH|iiii|JJJJ|kkkk|LLLL|mmmm|NNNN|
|302                               |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|ssTT|uuVV|wwww|wwXX|XXXy|
|501                               |    |    |    |    |    |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|
|556                               |    |__aa|BBcc|DDee|FFgg|HHii|JJkk|LLmm|NNoo|PPqq|RRss|TTuu|VVxx|xxxx|

|Script|Dvr Recording|

!|scenario           |Given a dvr with _ recorder|number|
|given dvr can record|@number                           |

!|scenario                  |and a season pass for _|seasonPass|
|when I create season passes|@seasonPass                       |

!|scenario                         |then the to do list should contain|expected|
|then the to do list should contain|@expected                                  |

![ script
Given a dvr with 1 recorder
and a season pass for cccccccc:200
and a season pass for FF:302
then the to do list should contain cccccccc:E:1-1
]!
{% endhighlight %}

This example is self-contained and can be written as a top-level, stand-alone page.

What follows is a breakdown of the page by its parts.
### Boilerplate Setup
The first three lines are traditionally on a top-level (hierarchy-wise) page. Since this example stands alone, it must say it uses slim, required fitnesse.jar (I'm using a Java example) and a path to my compiled Java code. I just got this code out of github, brought it up in eclipse and ran the unit tests before trying to write these FitNesse tests.

{% highlight terminal %}
!define TEST_SYSTEM {slim}
!path fitnesse.jar
!path /Users/schuchert/src/new_fitnesse_table_example/fitnesse-tutorials/DVR/bin

|import                     |
|com.om.example.dvr.fixtures|
{% endhighlight %}
Note that the import table must either be on this page or included in a SetUp page. Import tables are not otherwise inherited.

### Table Table to populate a program schedule
This next section populates a program schedule. The left-most column lists channel numbers. The cells across represent program names. Each program consists of a sequence of equal, case-sensitive letters. This example is copied from the [Tables Tables Tutorial](FitNesse.Tutorials.TableTables). This could be in a SetUp page or on the same page.

{% highlight terminal %}
|Table:Create One Day Program Guide|1:00|3/4/2008                                                        |
|                                  |1   |2   |3   |4   |5   |6   |7   |8   |9   |10  |11  |12  |13  |14  |
|200                               |aaaa|BBcc|cccc|ccDD|DDee|efff|ffff|fffg|gggg|gggh|hhii|jklm|nopq|rstt|
|247                               |aaaa|BBBB|cccc|DDDD|eeee|FFFF|gggg|HHHH|iiii|JJJJ|kkkk|LLLL|mmmm|NNNN|
|302                               |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|ssTT|uuVV|wwww|wwXX|XXXy|
|501                               |    |    |    |    |    |aaBB|ccDD|eeFF|ggHH|iiJJ|kkLL|mmNN|ooPP|qqRR|
|556                               |    |__aa|BBcc|DDee|FFgg|HHii|JJkk|LLmm|NNoo|PPqq|RRss|TTuu|VVxx|xxxx|
{% endhighlight %}

### Name the script table used by the following examples
Scenarios execute against a script. This simply names the script, an existing one from previous examples, to use for the following scenarios.

{% highlight terminal %}
|Script|Dvr Recording|
{% endhighlight %}

### Describe Scenarios Using New Placeholder Syntax
These scenarios use the placeholder syntax. Rather than alternating data cells with name cells, you can use _ to match elements. The matching is literal-string based, so watch out for things like extra or missing spaces.

Each of these are two lines, which is an artifact of using an existing fixture. The first line names a pattern. The second line refers to a function be found on some script table. The script table is called Dvr Recording, so there should be matching methods.

{% highlight terminal %}
!|scenario           |Given a dvr with _ recorder|number|
|given dvr can record|@number                           |

!|scenario                  |and a season pass for _|seasonPass|
|when I create season passes|@seasonPass                       |

!|scenario                         |then the to do list should contain|expected|
|then the to do list should contain|@expected                                  |

{% endhighlight %}

Here is the DvrRecording fixture used to execute these scenarios:
{% highlight java %}
package com.om.example.dvr.fixtures;

import java.util.Iterator;
import java.util.List;

import com.om.example.dvr.domain.Program;

public class DvrRecording {
   public void givenDvrCanRecord(int number) {
      CreateSeasonPassFor.getSeasonPassManager().setNumberOfRecorders(number);
   }

   public void whenICreateSeasonPasses(String listOfSeasonPasses) {
      String[] individualSeasonPasses = listOfSeasonPasses.split(",");

      for (String programNameChannel : individualSeasonPasses)
         addOneSeasonPass(programNameChannel);
   }

   private void addOneSeasonPass(String programNameChannel) {
      String[] parts = programNameChannel.split(":");

      String programName = parts[0];
      int channel = Integer.parseInt(parts[1]);

      new CreateSeasonPassFor(programName, channel);
   }

   public boolean thenTheToDoListShouldContain(String listOfEpisodes) {
      List<Program> toDoList = CreateSeasonPassFor.getSeasonPassManager()
            .toDoListContentsFor("");

      String[] episodesSets = listOfEpisodes.split(",");

      for (String episodeSet : episodesSets)
         if (!removeAllFrom(episodeSet, toDoList))
            return false;

      return toDoList.size() == 0;
   }

   private boolean removeAllFrom(String episodeSet, List<Program> toDoList) {
      String programName = extractProgramNameFrom(episodeSet);
      String baseEpisodeName = extractBaseNameFrom(episodeSet);
      int lowerRange = extractLowerRangeFrom(episodeSet);
      int upperRange = extractUpperRangeFrom(episodeSet);

      for (int episodeNumber = lowerRange; episodeNumber <= upperRange; ++episodeNumber)
         if (!remove(programName, baseEpisodeName, episodeNumber, toDoList))
            return false;

      return true;
   }

   private String extractProgramNameFrom(String episodeSet) {
      return episodeSet.split(":")[0];
   }

   private String extractBaseNameFrom(String episodeSet) {
      String[] values = episodeSet.split(":");

      String result = "";
      if (values.length > 1)
         result = values[1];

      return result;
   }

   private int extractLowerRangeFrom(String episodeSet) {
      String[] values = episodeSet.split(":");
      if (values.length > 2) {
         String lowRange = episodeSet.split(":")[2].split("-")[0];
         return Integer.parseInt(lowRange);
      }
      return Integer.MAX_VALUE;
   }

   private int extractUpperRangeFrom(String episodeSet) {
      String[] values = episodeSet.split(":");
      if (values.length > 2 && values[2].indexOf('-') > 0) {
         String highRange = episodeSet.split(":")[2].split("-")[1];
         return Integer.parseInt(highRange);
      }
      return 1;
   }

   private boolean remove(String programName, String baseEpisodeName, int episodeNumber,
         List<Program> toDoList) {
      String episodeName = String.format("%s%d", baseEpisodeName, episodeNumber);

      for (Iterator<Program> iter = toDoList.iterator(); iter.hasNext();) {
         if (matches(iter.next(), programName, episodeName)) {
            iter.remove();
            return true;
         }
      }
      return false;
   }

   private boolean matches(Program current, String programName, String episodeName) {
      return programName.equals(current.programName)
            && episodeName.equals(current.episodeName);
   }
}
{% endhighlight %}

### Create a script using new syntax
This new script syntax removes the need to use pipe characters to separate parts of a name from parameters. Instead, it is written more like a sentence and then matched against the scenarios above. The "![" syntax is a way to remove some of the cell formatting.

{% highlight terminal %}
![ script
Given a dvr with 1 recorder
and a season pass for cccccccc:200
and a season pass for FF:302
then the to do list should contain cccccccc:E:1-1
]!
{% endhighlight %}

### How Does It Work
There are two parts to this new feature:
* Placeholder syntax for scenarios
* Script table formatting

The second is superficial.

FitNesse sees: **Given a dvr with 1 recorder**, it notices that this matches the pattern: **Given a dvr with _ recorder**. It matches 1 to _, which is then assigned to one parameter named **number**. FitNesse replaces that pattern with the remaining lines of the script table (the one remaining), which gives:
{% highlight terminal %}
givenDvrCanRecord|@number                           |
{% endhighlight %}

The parameter @number is replaced with its assigned value 1. Next, FitNesse looks for a method named "givenDvrCanRecord" taking one parameter, an int, in the current script table, which is DvrRecording. This just so happens to match:
{% highlight java %}
   public void givenDvrCanRecord(int number) {
      CreateSeasonPassFor.getSeasonPassManager().setNumberOfRecorders(number);
   }
{% endhighlight %}

The remaining lines are matched, substituted and executed in a similar manner and the test executes.

This is not exactly correct. FitNesse does all of the matching and substituting. It then sends this processed test to Slim, which looks up the appropriate class, DvrRecording, and method. So all of the execution happens after all of the matching and substitution.

{% include nav prev="FitNesse.Tutorials" %}
