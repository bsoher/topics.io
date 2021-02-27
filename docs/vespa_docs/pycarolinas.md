---
sort: 8
---

# PyCarolinas Talk Outline (2012?)

1.  Title, background blurb, our names
	- 40 minute talk, so we need to define a few things and give a bit of context as to what (small) part of "academia" we're talking about today.

2. B - What is an NIH?
   - National Institutes of Health
   - $31 billion budget, 80% of which supports 50,000 grants at 2500 universities and institutions
   - In 2011, 62,000 research grants were submitted, 15% were funded.
   - Most researchers, once they get on the treadmill, spend most of their time trying to get the next grant.
   
3. B - What is 'academia'
   - It's a highly variable place, 
   - we are a research group inside a clinical Radiology department.
   - Many other departments, engineering, computer science, math and statistics, may have a different take on what I want to talk about today
   - I've been in 4 Radiology departments in my career, and I can say that most of what I'll present today applies to all of them.
   - We work with $3M machines, our work is linked to them, we tend not to do a lot of startups
     - This benefits open source in that we tend to be more collaborative
     - SPM is an example of 'free software' but charge for the manual ... ie consult on its use
   - Areas without $Millon boat anchors may actually have IP to consider, thus less likely for open source
   - Include it in the grant if you want to do an open source project
     - leverage with the university, grant *is* the contract with NIH

   
4. B - The Vespa Project - where did it come from
   - NIH grant for software maintainance and extension
     - tired of software projects shutting down when money tap shut off
     - willing to support existing software that showed longeveity and a community
   - 1996-2006 I helped create 2 software projects and worked with Jerry Matson who already had the third
   - Found that all three could be linked together to be greater than the whole - via sneakernet
     - Jerry's Matpulse program -> GAVA spectral simulation -> spectral fitting program. 
     - Tests theory against real results, allowed users to optimize for results before spending $$ on MR machine
   - Took 2 years to get funded. Grant awarded in 2008. Hired Philip 6 months in
   - Show pretty pics of our applications
     - but, what the apps do isn't the story today
     - today is about what we've learned while building these apps, what we tried and what did/didn't work

5. B - The Vespa Project - what need did it fill
   - standardized tools for data processing 
   - provenance, precise documentation of data processing to support scientific claims
   - extensible infrastructure for data processing methods - pipelines 
   - reusing the current data-handling framework - write once, use many

6. P - Inclusion of existing scientific code libraries within a Python-based infrastructure
   - Python & SWIG made reusing GAMMA realistic. It restored GAMMA to relevance
   - HLSVDPRO - easy to call from Python

7. P - Cross-platform acrobatics: three Pythons, three platforms, 32- or 64-bit, one community
   - Eight libraries
   - Different languages
   - Different sources
   - "B" biggest issue for me is cost of maintenance after grant is done

8. B - To GUI or not to GUI ... of course they all want GUI
   - Lots of initial work is interactive, optimize processing to get results we want
   - may not have a "measure of goodness", results are subjective need to visualize
   - marketing to get people to try the software

9. B - Increasing expectations for distributed computing across cores, CPUs and clusters
   - eventually, there may be a lot of data to process or re-process as new methods are found
   - cluster computing is widespread, but not easily applied
   - MR data and spectral simulation is trivially separable and can benefit from distributed processing
   - people still like it to "just work" may not understand how to use interactively (IPython)

10. P - Less is more, or How to make software appear professional at the cutting edge of science
   - "B" most researchers like the idea of professional look, but don't have the discipline to maintain
   - "B" always trying new things and maybe take shortcuts in coding and GUI to try them then never reove.

11. B - Documentation, online resources, tools for building community
