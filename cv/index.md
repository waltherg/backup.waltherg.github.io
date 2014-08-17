---
layout: default
title: "Georg Walther CV"
date: 2014-08-16
tags: cv
---

# Georg Walther

<center>
    <span class="glyphicon glyphicon-home"></span><a href="http://georg.io"> http://georg.io    </a>
    <span class="fa fa-linkedin"></span><a href="http://uk.linkedin.com/in/georgwalther"> http://uk.linkedin.com/in/georgwalther </a><br/>
    <span class="glyphicon glyphicon-envelope"></span><a href="mailto:contact@georg.io"> contact@georg.io    </a>
    <span class="fa fa-stack-overflow"></span><a href="http://careers.stackoverflow.com/georgwalther"> http://careers.stackoverflow.com/georgwalther </a>
  </div>
</center>

## Summary

I am a biochemist \& computational biologist by training,
have worked as a software engineer for a startup, and
am now in training to become a data scientist.

On the technical side I am enthused by data analysis, predictive analytics, and engineering clean code.
I am also enthusiastic about understanding the business opportunities that data-driven decision making can produce.

***

## Work Experience

### Data Scientist at Singular Intelligence (2014), London, United Kingdom

As part of the Science 2 Data Science summer school I did work for Singular Intelligence, a startup that helps businesses develop data- and analytics-driven corporate strategies.

As part of my work I did data exploration and visualization in Python with pandas and matplotlib.
I further employed scikit-learn (linear models and random forests) to develop a predictive model of sales performance as a function of business-relevant features.

### Software Engineer at WalletSaver (2014), Rome, Italy

WalletSaver is a startup in the space of personalized and data-driven recommendation systems.
WalletSaver develop an app that measures securely parameters of mobile phone usage and recommends the best mobile phone plan based on cost, signal coverage, and phone carriers called.

I engineered the recommendation engine behind WalletSaver in Python with SQLAlchemy (a SQL object-relation mapper library) and Memcached to speed up data queries.
I followed a test-driven development methodology for this project and performed timing-based optimization such as precomputing certain data.
I sought actively customer feedback and amended the recommendation engine in response to feedback.

I also engineered and maintained geolocation code, critical for signal coverage analysis, in Python with the requests library.

### Teaching Assistant at ETH Zurich (2008 - 2009), Zurich, Switzerland

Research and preparation of lecture notes, preparation of exercise questions, preparation of exam questions, grading exams.

***

## Education and Certificates

### Science 2 Data Science (August 2014 - September 2014), London, United Kingdom

[Science 2 Data Science (S2DS)](http://www.s2ds.org/) is an industry-sponsored summer school that leads graduates with numerical backgrounds into the field of data science.

At this course I learned about Hadoop, SQL databases, NoSQL databases, natural language processng, statistics with R, machine learning with scikit-learn, and data visualization.

I further learned about business opportunities that arise from data-enhanced and data-driven business models, economics, marketing, finance, project management, and corporate strategy.

I summarize my experience at this school at [http://georg.io/s2ds](/s2ds).

### PhD Researcher (September 2010 - September 2014), John Innes Centre, Norwich, United Kingdom

My research evolved around modeling cell polarity under a number of complex conditions.
The formation of poles (e.g. front v. back) is important for cells to know which direction to grow or migrate. Specifically, I studied these equations in extreme environments such as exceptionally large and small cells.

I wrote unit tested code in Python with NetworkX that implements a mathematical framework for bifurcation analysis of large reaction networks.
Here I had to rephrase a number of combinatorial problems as classical graph-theoretic problems to reduce computational cost.
One particular graph-theoretic algorithm, the enumeration of all cliques, was not available in NetworkX at the time.
I implemented a published algorithm for this problem and my implementation has since been added to NetworkX.

I further implemented variants of Crank-Nicolson in Python with NumPy to integrate numerically my specific convection-diffusion-reaction systems.
I performed timing-supported optimization of my code and added inline C code where necessary for further speed improvement.

During my PhD I also wrote numerical simulations of a high-dimensional dynamical system in pure C with support of the GNU Scientific Library.

### Visiting Scholar (2009 - 2010), The University of British Columbia, Vancouver, Canada

Research and write up of Master thesis: Stochastic Simulations of Cell Polarization and Wave Pinning.

For this project I implemented Gillespie's stochastic simulation algorithm where I wrote the simulation setup in MATLAB and time-critical code in C.
For the same project I made extensive use of numerical bifurcation packages such as XPP/auto and Matcont.

### Master of Science (2010), ETH Zurich, Zurich, Switzerland

Self-designed major in computational biology and biological chemistry with strong focus on numerical analysis and data analytics.

Specific mathematical and computational courses that were part of my curriculum:

* Bioinformatics in-depth & Computational Systems Biology,
* Numerical Mathematics for CSE,
* Software Architecture,
* Probability and Statistics,
* Data Structures & Algorithms,
* Compiler Design I, and
* Computational Statistics.

### Bachelor of Science (2008), ETH Zurich, Switzerland

Major in biochemistry.

***

## Skills

<table class="table table-striped">
  <thead>
    <tr>
      <th>Type</th>
      <th>Name</th>
      <th>Skills</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Programming</td>
      <td>Python</td>
      <td>Avanced, numerical simulations, database-driven backend development, data analysis, predictive analytics (research projects and work for startups)
    </tr>
    <tr>
      <td></td>
      <td>C</td>
      <td>Avanced, numerical simulations, stochastic simulations (research projects)</td>
    </tr>
    <tr>
      <td></td>
      <td>C++</td>
      <td>Familiar and learning (education and self-taught)</td>
    </tr>
    <tr>
      <td>Operating Systems</td>
      <td>Linux, Windows</td>
      <td>Advanced user, Python package development and distribution, scripting</td>
    </tr>
    <tr>
      <td>Databases</td>
      <td>PostgreSQL, MySQL</td>
      <td>Working knowledge, data modeling, querying, and data integration into time-critical code (work for a startup)</td>
    </tr>
    <tr>
      <td></td>
      <td>Neo4j, Rexster</td>
      <td>Working knowledge (personal side projects)</td>
    </tr>
    <tr>
      <td>Project Management and Collaboration</td>
      <td>git</td>
      <td>Advanced, have worked on shared codebase in a startup</td>
    </tr>
    <tr>
      <td>Communication</td>
      <td>Work communication</td>
      <td>I am comfortable presenting my work in informal and formal settings</td>
    </tr>
    <tr>
      <td></td>
      <td>Scientific writing</td>
      <td>I have authored two scientific articles as lead author</td>
    </tr>
    <tr>
      <td></td>
      <td>Technical writing</td>
      <td>I write technical articles about my side projects at <a href="http://georg.io">http://georg.io</a></td>
    </tr>
  </tbody>
</table>

***

## Publications and Software

### GraTeLPy: Graph-Theoretic Linear Stability Analysis

**Walther GR, Hartley M, Mincheva M (2014): GraTeLPy: Graph-Theoretic Linear Stability Analysis, BMC Systems Biology**
(software available at
[http://github.com/gratelpy](http://github.com/gratelpy))

I implemented a graph-theoretic framework for the analysis of dynamical
systems in Python with NetworkX.
I designed the algorithm, implemented it along with auxiliary methods that were necessary, and combined graph-theoretic approaches in novel ways to reduce combinatorial blowup.

I further responded to user feedback with extensive testing on Windows, OSX, and Linux and simplification of the installation process.

### Deterministic Versus Stochastic Cell Polarisation Through Wave-Pinning

**Walther GR, Marée AFM, Edelstein-Keshet L, Grieneisen VA (2012): Deterministic Versus Stochastic Cell Polarisation Through Wave-Pinning, Bulletin of Mathematical Biology**

I implemented Gillespie's stochastic simulation algorithm in MATLAB and time-critical parts of the code in C.
I further made extensive use of numerical bifurcation packages XPP/auto and MatCont.

***
