---
title: "EDAnalyzers"
teaching: 7
exercises: 1
questions:
- "What is an EDAnalyzer and what does it contain?"
- "What files are relevant in a EDAnalyzer?"
objectives:
- "Learn what an EDAnalyzer is and how it is structured"
- "Learn what c++, python and xml files are relevant."

keypoints:
- "An EDAnalyzer is a an edm class that generates a template for any analysis code using CMSSW."
- "There are essentially three important files in an EDAnalyzer package, the source code in c++, the python config file and a Buildfile for tracking dependencies."
---

## Structure

First, make sure you start up your VM or container like it was discussed in the previous episode.

EDAnalyzers are modules that allow read-only access to the Event. They are useful to produce histograms, reports, statistics, etc.  Take a look at the `DemoAnalyzer` package that we created while validating our CMSSW working environment; it is an example of an EDAnalyzer.  

Go to your `CMSSW_5_3_32/src` area and make sure you issue the `cmsenv` command (if you are in the same session as the previous episode, there is no need to issue the command again but it never hurts doing it again):

~~~
cd <mybasepath>/CMSSW_5_3_32/src
cmsenv
~~~
{: .language-bash}

Let's explore the DemoAnalyzer package:

~~~
ls Demo/DemoAnalyzer/
~~~
{: .language-bash}

~~~
BuildFile.xml  demoanalyzer_cfg.py  doc  interface  python  src  test
~~~
{: .output}

~~~
ls Demo/DemoAnalyzer/src
~~~
{: .language-bash}

~~~
DemoAnalyzer.cc
~~~
{: .output}

Note that it has a similar structure as any of the CMSSW packages we mentioned [before](../01-introduction/index.html#structure-and-architecture).  In this sense, our `DemoAnalyzer` is just one more CMSSW package.  However, the headers and implementation of our simple DemoAnalyzer are coded in one single file under the `src` directory.  The file was automatically named `DemoAnalyzer.cc`

> CMSSW could be very pick about the structure of its packages.  Most of the time, scripts or other tools expect to have a `Package/Sub-Package` structure, just like our `Demo/DemoAnalyzer` example.
{: .testimonial}

We also notice we have a python configuration file called `demoanalyzer_cfg.py` (unlike its cousins, it is not inside the `python` directory).  This is the default configurator for the `DemoAnalyzer.cc` code.

Finally, there is a `BuildFile.xml`, where we can include any dependencies if needed.

All EDAnalyzers are created equal; of course, if made with the same `mkedanlzr`, they will look identical.  The `DemoAnalyzer.cc` is a skeleton, written in C++, that contains all the basic ingredients to use CMSSW libraries.  So, in order to perform a physics analysis, and extract information from our CMS open data, we just need to understand what to add to this code and how to configure it.


{% include links.md %}
