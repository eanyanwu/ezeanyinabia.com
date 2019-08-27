---
layout: post
title: Service Fabric
excerpt_separator: <!--more-->
---

Service Fabric is the foundation of the analysis system we have built up at work. I never spent enough time learning what it is and how it works. I suspect this has been a hidden barrier to what I could contribute in that part of the code. This is my attempt to break it down by learning as much as I can about Microsoft Service Fabric. 


<!--more-->

# Contents
1. [Intro](#intro)


# Intro

The genesis of the "Service Fabric" idea started in early 2000s when Microsoft was trying to build an eHome plarform. The idea was that since there was already a fair number of Microsoft products in the home at that point, these products could form peer-to-peer networks that would be an extension of Microsoft's real network. Getting this to work required a federated system that would be composed of multiple machines. This sytem would have to be able to answer questions such as: who is a part of the system?, when did member A join? which machines are down? This is a distributed computing problem.

Fast-forward to present time. Service Fabric is a reality. It tries to create an abstraction over distributed computing and give the illusion that programming for one process is similar to programming for 1000s of distributed processes, whether they be stateful or stateless. This is a powerful abstraction because I have the eerie feeling that distributed programming is hard. It would be great not to have to worry about replication and consistency of my data whether the process responsible for saving that data is running on one machine or multiple. 


Source:  
[Chat with Gopal Kakivay, who was at the time VP of Microsoft's Hyperscal Compute Team, Home of Service Fabric](https://www.youtube.com/watch?v=MrfcP6dS6mU)

