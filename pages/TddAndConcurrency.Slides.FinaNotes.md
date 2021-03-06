---
title: TddAndConcurrency.Slides.FinaNotes
---
{% include nav prev="TddAndConcurrency.Slides.KeepItAwayFromMe" up="TddAndConcurrency.Slides" next="TddAndConcurrency.Slides.RandomStuff" %}

## Final Notes

### Hints
* Run with more threads than processors
* Run on the target platform when tuning
* Run with the –server VM argument
* Rely on published algorithms when possible
  * Producer/consumer
  * Readers/writers
  * Dining Philosophers
* Don’t build your own thread-safe containers
  * Use java.concurrent
  * JDK 1.4, use http://g.oswego.edu/dl/classes/collections/
  * http://blogs.azulsystems.com/cliff/2007/03/a_nonblocking_h.html
* Consider using locks if in Java 5 or stick with intrinsic locks if moving to 6.
* There’s a lot of gold in java.lang.concurrent!

{% include nav prev="TddAndConcurrency.Slides.KeepItAwayFromMe" up="TddAndConcurrency.Slides" next="TddAndConcurrency.Slides.RandomStuff" %}
