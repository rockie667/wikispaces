---
title: RhinoMocks.setup
---
* Download and install [NUnit](http://www.nunit.org/index.php?p=download).
* Add a reference to NUnit in you project: Project:Add Reference (.NET Tab), search for NUnit.framework and add it.
* [Download Rhino from here.](http://www.ayende.com/projects/rhino-mocks/downloads.aspx). Note you'll need to pick a version that matches the version of .Net your project uses.
* Extract the zip somewhere (I used c:\libs\Rhino)
* Add a Reference to your project: Project:Add Reference (Browse Tab), search for the Rhino.Mocks.dll and add it.
* Here is a stand-alone example showing how to do a few common things in Rhino: [Basic Usage Example](RhinoMock.examples.BasicUsage).
* Here is a simple example that tests a LoginService with all of the dependencies left as interfaces: [Sample Login Application](RhinoMocks.examples.SampleLoginApplication).

