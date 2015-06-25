---
layout: post
title: "Continues Integration Performance Testing"
excerpt: "Jmeter, Jenkins, Taurus and Blazemeter reporting to help with continues integration performance testing"
modified: 2015-06-21
categories: tech
tags: [jmeter, jenkins, taurus, blazemeter]
image:
  feature:
comments: true
date: 2015-06-21T00:00:00-00:00
---

Discovering performance problems as early as possible would save you money and time. How about finding them out right after a commit?
Changing environment settings? How could you be sure that all 20+ nodes of your application are healthy running your changes? Some manual testing or one thread automated testing simply wouldn't do it. How about generate some smoke load?

To solve the challenges above and even more, you can add Smoke and Benchmark performance tests to your continues integration workflow.
To do so I used:

* [Jenkins][jenkins] as a continues integration (CI) agent
* [Jmeter][jmeter] for load and performance testing
* [Taurus][taurus] to orchestrate Jmeter in Jenkins
* [Blazemeter][blazemeter] for tests reporting

Nice part about the tools - they are all free.

#### Prerequisites
1. Configure continues deployment of code to environments.
2. Create a [Jmeter][jmeter] script to cover your key functionality: add assertions to all requests, parameterise the script with all possible input values (it's not manual testing, you can literally add all 10K search strings that your customers are using in production), make it real world like with addition of probabilities of each of the request.

#### Configuring performance test in CI
Now let's add performance tests to our CI: we want to run smoke load test with a few virtual users after each build and run benchmark test for every release build.

1. Install Taurus.
[Taurus][taurus] is an open source tool to configure and run different load/performance test setups. Single Jmeter script can be used to run various types of tests: ramp-up, smoke, benchmark etc. [Installation instructions][taurusinst].

2. Create [configuration yml][taurusconfig] files for each of the environment and test type. Check example below.

3. *Run test* with command:```bzt smoke.yml```

4. Configure Jenkins job to run test on each  builds. Jenkins freestyle project can be used. 

###### Taurus Yml file example for a smoke test ([example gist][example])
{% highlight yaml %}
#! /usr/local/bin/bzt
 ---
 execution:
   concurrency: 10
   ramp-up: 30s
   hold-for: 5m
   scenario:
     script: Smoke_script.jmx
     properties:
         my-hostname: YOUR_HOST.COM
  
 reporting:
   - module: blazemeter
     test: Smoke Test
   - module: junit-xml
     filename: report.xml
     data-source: pass-fail
   - module: fail-criteria
     criterias:
     - avg-rt>250ms for 30s, continue as failed
     - failures>5% for 5s, continue as failed
     - failures>50% for 10s, stop as failed
     - rc5??>0 for 10s, continue as failed
     
 modules:
   blazemeter:
     token: Your Blazemeter Token
     browser-open: false
   console:
     disable: true
     
 settings:
   check-interval: 5
{% endhighlight %}


 
Sweet part that now you can use one [Jmeter][jmeter] script.jmx for all environments and test types: all you need is to change hostname and runtime settings like number of concurrent users in the [yml configuration][taurusconfig].

Enjoy testing!

#### Notes

###### What Taurus is Doing?
When you running Taurus (bzt) it: 

* overrides settings in the Jmeter script and creates a new one in the test directory (name will be prompted to the console);
* starts [Jmeter][jmeter] in a non-UI mode using the new script;
* disables some of the plugins specified in the script like Loadsophia plugin;
* sends report data to [Blazemeter][blazemeter] in a real time, so you can analyze and share it on the fly;

###### Yml Options
Lets walk through some of the options in the yml file:

* *execution* is mainly used by Taurus to override your script settings in the Jmeter script ["Thread group"][threadgroup] settings.
Settings that are not specified in the section will be taken from Jmeter script.
* *properties* is custom parameters. In the Jmeter script you can use with [property functions][jmeterproperty]: ```${__P(my-hostname,YOUR.DEFAULT.HOST.COM)}```

More about options in [Taurus docs][taurusconfig].

###### Reporting
* *reporting module blazemeter* adds test data to the [Blazemeter][blazemeter] account reporting specified in the test section. Your API Key can be found under profile settings. You might as well use anonymous reporting, but you will lose the feature I'm enjoying the most: comparison trends for response times, errors and hits/sec for different builds:

<figure>
	<a href="https://docs.blazemeter.com/customer/portal/articles/1742684-load-results-report"><img src="/images/2015-06-21_taurus_jmeter/blazemeter-reporting.jpg" alt="image"></a>
	<figcaption>Blazemeter reporting trend for series of Smoke tests.</figcaption>
</figure>

Another nice trend you can configure right in the [Jenkins][jenkins] with [junit plugin][junit] and Taurus *module junit-xml* to report your criterias status to xml file. Criterias can be specified in Taurus configuration yml, check the config above.

<figure>
	<a href="https://wiki.jenkins-ci.org/display/JENKINS/JUnit+graph"><img src="/images/2015-06-21_taurus_jmeter/junit-reporting.jpg" alt="image"></a>
	<figcaption>Jenkins Junit reporting trend for series of Smoke tests.</figcaption>
</figure>


You could do much more with Taurus and Jenkins. Check following [Taurus + Jenkins Youtube tutorial][tutorial].

[taurus]:  https://github.com/Blazemeter/taurus
[taurusinst]: https://github.com/Blazemeter/taurus/blob/master/docs/Installation.md
[taurusconfig]: https://github.com/Blazemeter/taurus/blob/master/docs/ConfigSyntax.md
[reporting]:  https://github.com/Blazemeter/taurus/blob/master/docs/Reporting.md
[blazemeter]: https://blazemeter.com
[example]: https://gist.github.com/Nefry/a758ef8088a6a003061a
[jmeter]: http://jmeter.apache.org/
[threadgroup]: http://jmeter.apache.org/usermanual/test_plan.html#thread_group
[jmeterproperty]: http://jmeter.apache.org/usermanual/functions.html#__P
[jenkins]: https://jenkins-ci.org/
[junit]: https://wiki.jenkins-ci.org/display/JENKINS/JUnit+Plugin
[tutorial]: https://www.youtube.com/watch?v=QuY0Qcdd90A
