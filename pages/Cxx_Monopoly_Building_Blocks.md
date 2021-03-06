---
title: Cxx_Monopoly_Building_Blocks
---
All of these examples run under cygwin. In addition to cygwin, these examples use the boost library (which can be installed as part of cygwin) and cxxunit, which you can download.

## Building
For these examples I'm using make. The Boost C++ libraries recommend using bjam, however, in the spirit of using the simplest thing that can possibly work, here is a trivial makefile that rebuilds the world each time;
{% highlight terminal %}
CxxTest_DIR = ~/cpp/cxxtest/cxxtest

all:
	$(CxxTest_DIR)/cxxtestgen.pl *Test.hpp -o tests.cpp
	g++ *.cpp -o tests -I /usr/include/boost-1_33_1 -I $(CxxTest_DIR) /usr/lib/libboost_regex-gcc-mt-1_33_1.a
	./tests
{% endhighlight %}

### **Warning**
This is not meant to serve as a makefile for a "real" system. It has all of the necessary characteristics:
* It's easy, you type make and it does **everything**
* If you system is small, it's fast. For the TDD example, it takes very little time to perform a full build and run all of the tests (after iteration 2 it took about 7 - 6 seconds on my laptop)
* It's reliable. If there's a problem building or the tests don't pass, it's because I messed up, not because the build doesn't work.

Here's what you'll need to do to get this to work for you:
* Install cygwin with the development package
* Download and install cxxtest
* Update the CxxTest_DIR macro to point to where you extracted the CxxTest installation
* Create a directory and in that directory create a file called "makefile" and paste the contents above with the updated CxxTest_DIR macro
* Create each of the following examples, noting the name of the files.

## Examples
These examples use CxxTest as a way to write tests to understand how to use parts of boost and the C++ standard library. Starting with these examples, you'll hopefully be able to continue experimentation on your own.

----
### Main.cpp
You'll need this as a driver. It's your main function:
{% highlight cpp %}
#include <cxxtest/ErrorPrinter.h>

int main(int argc, char* argv[]) {
	CxxTest::ErrorPrinter errorPrinter;
	errorPrinter.run();
	
	return 0;
}
{% endhighlight %}

----
### IStringStreamTest.hpp
{% highlight cpp %}
#include <cxxtest/TestSuite.h>

#include <iostream>
#include <string>

using namespace std;

class IStringStreamTest : public CxxTest::TestSuite {
public:
	void testReadingFourLinesFromStringStreamAroundString() {
		string s("hello world\nfoo\nbar\nbaz\n");
		istringstream stream(s, istringstream::in);

		string line;
		getline(stream, line);
		TS_ASSERT_EQUALS("hello world", line);
		getline(stream, line);
		TS_ASSERT_EQUALS("foo", line);
		getline(stream, line);
		TS_ASSERT_EQUALS("bar", line);
		getline(stream, line);
		TS_ASSERT_EQUALS("baz", line);
		getline(stream, line);
		TS_ASSERT_EQUALS("", line);
		TS_ASSERT(stream.eof());
	}
};
{% endhighlight %}

----
### BoostRegexTest.hpp
{% highlight cpp %}
#include <cxxtest/TestSuite.h>

#include <boost/regex.hpp>

using boost::regex;
using boost::regex_match;

class BoostRegexTest : public CxxTest::TestSuite {
public:
	void testCommentLine() {
		regex re("\\`[ \t]*#.*\\'");
		
		TS_ASSERT(regex_match("# ---------", re));
		TS_ASSERT(regex_match("\t# ---------", re));
		TS_ASSERT(regex_match("  # ---------", re));
		TS_ASSERT(regex_match(" \t  # ---------", re));
		TS_ASSERT(!regex_match("go\t", re));
	}

	void testBlankLine() {
		regex re("\\`[ \t]*\\'");

		TS_ASSERT(regex_match("", re));
		TS_ASSERT(regex_match("          ", re));
		TS_ASSERT(regex_match("\t\t\t\t", re));
		TS_ASSERT(regex_match(" \t \t", re));
		TS_ASSERT(!regex_match("a", re));
		TS_ASSERT(!regex_match("   \ta", re));	
	}
};
{% endhighlight %}

----
### BoostTokenizerTest
{% highlight cpp %}
#include <cxxtest/TestSuite.h>

#include <boost/tokenizer.hpp>
#include <string>
using std::string;

class BoostTokenizerTest : public CxxTest::TestSuite {
typedef boost::tokenizer<boost::char_separator<char> > tokenizer;

public:
	void testMatchingTabsAsDelimiters() {
		string s = "go\tGo\nlocation\tMed Ave\n";
		boost::char_separator<char> tabNl("\t\n");

		tokenizer tok(s, tabNl);
		tokenizer::iterator it = tok.begin();

		TS_ASSERT_EQUALS(*it, string("go"));
		++it;
		TS_ASSERT_EQUALS(*it, string("Go"));
		++it;
		TS_ASSERT_EQUALS(*it, string("location"));
		++it;
		TS_ASSERT_EQUALS(*it, string("Med Ave"));
		++it;
		TS_ASSERT_EQUALS(it, tok.end());
	}
};
{% endhighlight %}
