---
title: JPA_Tutorial_4_Introduction
---
So far, every query we've used has returned a single kind of result. In this tutorial we change things up a bit by moving from only using Books in our Library to working with Books and Dvd's, both of which inherit from **Resource**. When we search for a **Resource** we might get back Books, Dvd's, or both. This makes our queries polymorphic.

We pick up at the end of [JPA_Tutorial_3_A_Mini_Application](JPA_Tutorial_3_A_Mini_Application) and perform two major transformations:
* Introduce the Resource class and make the appropriate refactorings
* Introduce a new type, Dvd
