---
author: mhallwac
category: practices
filename: 2018-01-25-experience-as-an-intern.md
layout: post
tagline: Reflection on experience gained through the internship
title: Experience as a Quality Assurance Intern
tags: 'QA, experience, intern'
---
The following is detailed review of my experience as a Quality Assurance intern for the Digital Library Technology team at the Hesburgh Library, Notre Dame.

## First Two Weeks
Having never used Ruby before and having only done manual testing for websites, the task of learning Ruby and its uses for automated testing was daunting. It was difficult to determine the best way to learn Ruby while also learning how the website functions from front end to back end in order to properly test its functionality.

After a couple of trial days, Harsh, my supervisor and mentor, and I decided it would be best if I spent the morning learning Ruby through [Code Academy](https://www.codecademy.com/learn) and in the afternoons, we would dive into some front end test cases. After another week of this back and forth learning schedule, I was finally ready to start writing my own test cases.

## The Three Testing Environments
The best part of this internship was that it was not boxed into just one of the testing environments.  Throughout the summer, I worked on functional, integration, and unit testing which gave me quite a broad view of how website development is organized. In addition, the sequence with which I learned these environments helped with understanding how each environment relates to the other from the front-end to the back-end.

### <span style="color:green"> 1. Functional Testing</span>
When I first started diving into functional testing, I created use cases for the site I was working on by manually exploring the site. After the use cases were created, I started writing the automated specs for the site.  This process still had a learning curve in the beginning because while I had become familiar with Ruby, I still had to explore the methods associated with the Capybara gem (a great cheat sheet for learning these methods can be found [here](https://gist.github.com/zhengjia/428105).

My first test case was a sloppy, bulky scenario that was quite hard to understand what exactly was being tested on the web page.  However, within about a week, I started learning more about the conventions of code readability and was able to whittle down the code of the scenarios to about 5-6 lines.

### <span style="color:blue"> 2. Integration Testing</span>
Next up, learning how to test API integration.  While these test cases were still written in Ruby, the integration test cases had much more depth to them due to the Swagger definition associated with each API. Using these Swagger definitions, I needed to make sure that the API response had all the proper keys that were defined in the Swagger definition.  In doing so, I create a ResponseValidator class with one method that would get the expected keys from the Swagger definition file (needed to create this method since the resolve_refs JSON method did not work properly) and one method that parsed through the JSON of the API response to find the list of response keys. However, these methods took time to perfect as they worked for some of the APIs but not others.  After a period of trial and error, I finally found a [recursive way](https://github.com/ndlib/QA_tests/commit/0dc77b2acb1356323e90657454c92f858dd0b261) to dig to the lowest level of each schema in the Swagger definition and API response that worked for every API.  While digging through every minute detail of multiple Swagger files was by no means a walk in the park, after finishing the ResponseValidator class, I had a much better understanding of how APIs connect the front-end to the back-end of the website.

### <span style="color:red"> 3. Unit Testing</span>
While unit testing is usually done by the developers, Harsh organized for the Web Development team to work with me in learning how to test that the back-end code of the website was functioning properly. It was difficult to get started since I had never worked with Javascript before, so I spent one day reading through the examples of what I would be testing.

I learned that, in a broad sense, Unit Testing was essentially throwing each method parameters for the various scenarios and making sure they were producing the expected results (E.G. not rendering when given improper parameters). While it is clearly more complicated than that, this point of view helped with not getting lost in the complexity of the back-end Javascript.

## Why This Sequence Helped
By the end of the summer, I had not only learned how to write automated tests for each testing environment, I had also gained insight into how each facet of the website is connected.  I largely attribute this insight to the order with which I was rotated through each environment.  By starting off with functional testing, I learned how the front-end of the website functioned. By moving into integration testing next, I learned how APIs function and connect the front-end to the back-end. Finally, ending with unit testing allowed me to further understand how each facet of the website was designed to work on the back-end and tied together the three environments of automated testing.

## Concluding Remarks
Before this internship, I had no experience in web development languages such as HTML/CSS, Javascript, etc.  Most of my prior knowledge was creating apps and games in Java and Visual Basic for Applications.  So when I saw the job posting, I realized that I had this gap of knowledge when it came to web development and testing.  While my undergraduate major is in Information Technology and Management (ITM), I thought that a broader knowledge of coding would not only aid in the job search after graduation but would also help open up different areas of ITM which are more code-oriented.  As it turns out, I was right but in more ways that I had originally thought.

What I learned from this internship has not only made me more knowledgeable in web development but has constantly given me a leg up in my ITM classes as well.  While others were struggling to understand concepts such as Agile development, using Github for code sharing, and the conventions of Unix, I had real world knowledge of these concepts.  It is fairly common for interns to walk out of an internship and ask themselves "What did I really learn?", but I was constantly reminded of exactly what I learned because it was applicable to so many different situations and classes.

While I was focused on becoming a better coder by learning new languages and learning how to test websites in different environments, the structure of my internship (i.e. sprints, stand-ups, Jira) made me a better employee by teaching me to how to collaborate and organize within a team environment; something which will allow me to be a better employee and teammate in the future.

While I still have a lot to learn about website development and maintenance, as a QA Intern for Hesburgh Libraries, I was given a foundation of knowledge that will carry with me throughout my career.
