---
title: Moq
---
In May of 2009 I worked with a group in Canada that was using Moq so I got my first exposure to that library. As far as mocking libraries go, it's one of my favorite (for statically-typed languages). It takes advantage of both Generics as well as Lambdas, so it's well designed, uses contemporary features and seems to work rather well.

||
[[http://blog.objectmentor.com/articles/2009/05/19/a-first-look-at-moq#comments|First Object Mentor Blog Entry]]||This is a copy of my first blog on the subject.||
||[[http://blog.objectmentor.com/articles/2009/05/22/strict-mocks-and-characterization-tests|Using Strict Mocks to assist in Characterization Tests]]||A second blog article related to Moq.||

The first blog entry documented some of the work I did to make sure I understood the library before working with the group. I then had the group repeat the work with a [[Tdd.Problems.LoggingIn|loose set of requirements]]. We worked on this together and this is what we ended up with:
>> [[Moq.Logging In Example Implemented|Logging In Example in Moq]].