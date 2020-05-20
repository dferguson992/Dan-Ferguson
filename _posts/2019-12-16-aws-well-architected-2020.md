---
layout: posts
title: AWS Well-Architected 2020
permalink: /posts/aws-well-architected-2020
published: true
---

Ippon USA has been working to lock in Well-Architected status in the North America region for the last six months.  By the end of 2019, Ippon USA will join Ippon France in meeting the [Well-Architected program requirements](https://aws.amazon.com/partners/find/partnerdetails/?n=Ippon%20Technologies&id=0010L00001iWx29QAC&t=psf-overview) for two consecutive quarters.  Over the last 6 months, the WAR (Well-Architected Review) team at Ippon has learned through experience.  

This past week, I was lucky enough to attend re:Invent 2019 where I learned other AWS Partners have had similar experiences.  In addition to the new announcements surrounding Well-Architected, there's a lot to think about for how Ippon will use the program in 2020.

## Well Architected is a Mindset
The 5 pillars of Well Architected are often called "AWS Tribal Knowledge."  But the principles that define these pillars are not proprietary AWS concepts.  The strategies and techniques used to build performant, reliable, and secure applications which are easy to maintain and inexpensive have been well established for decades.  So why is the Well-Architected program so valuable to clients?  Based on what I learned at re:Invent 2019, it seems like Well Architected is not valuable to clients unless they truly practice Well-Architected at every level of their organization.  

The most successful AWS Partners in the Well-Architected (WA) space ensure all of their employees, including sales, are well-versed in the WA Framework.  This company wide adoption allowed one of the speakers at re:Invent to conduct over 200 WA reviews in 2019, with a rate of rollover into additional business.  A similar company touted the same number of reviews, but expressed difficulty in converting reviews into additional business.  This is not to say the architects at Company A are better at reviewing than the architects at Company B.  The big difference between the two organizations is how they treated the 5 pillars of WA internally.  Company A said they dove into WA head-first, while Company B implied they were testing the program out.  This hesitance is evidenced by the niche group of WA focused employees at Company B, compared to the front-to-back adoption of WA at Company A.  

When your entire organization adopts WA principles, these common sense tenants that any seasoned engineer would claim to practice on a daily basis anyway, the organization will retain new business as a consequence.

## Well-Architected is a Sales Tool
The WA framework is a great addition to a sales pitch for new clients.  Ippon Technologies's motto is "Discovery to Delivery."  This usually means initial engagements with a client start with a "discovery" period where we interview engineers and product owners, define deliverables, build a project timeline and crank out as many relevant demos as possible.  After this process, which can be lengthy depending on the proposed scope, we hit the ground on "delivery," usually in an agile project management style.  

Before re:Invent, we never rolled WA reviews into the "discover" phase of our engagements.  Now, we will start introducing WA reviews as part of the initial assessment.  By conducting a review early on, it becomes clear to all parties exactly what the rest of the assessment needs to focus on.  The scope of the engagement is well-defined within the confines of the review process.  In addition to raising awareness for the technical flaws which need to be remedied during the engagement, the WA review also raises awareness for larger organizational issues that could be addressed during the engagement.  

For example, if our client is looking to add a new feature to their app before they sign contracts for new business in a different region, the Operational Excellence, Performance, and Reliability pillars should be used to assess if the client is ready to scale their business as they have planned.  Armed with this knowledge, Ippon engineers can be ready to scope DR replication, CDS deployments, and even a hiring process for new engineers on behalf of the client.  Just by allocating three or four hours at the beginning of the "discovery" phase to have a focused discussion on WA concepts, we will be able to streamline "discovery" initiatives and deliver a stronger product by the end of the engagement.

## Well-Architected is not an Innovation Driver
After re:Invent 2019, I am so eager to use all of the shiny new features that can be found on AWS.  Some of this features can be quite expensive though, and can introduce complexity into an otherwise simple problem.  This creates solutions which are not Well-Architected.  My last, and most important, note on WA at Ippon in 2020 is that Well-Architected is not going to be an excuse to introduce new, innovative tools and services for the sake of innovating.  

One of the presenters at the WA event shared story after story of how when they started their journey with the WA program, the temptation to recommend cutting edge tools and services on AWS was too great to resist.  This created technical debt in the recommendations that rolled into an operational cost for their engineering staff.  I am happy to say, the partner quickly remedied this practice internally.  Now, the remediations they suggest include innovative tools and services when and where they are required.

It may be difficult to assess what could be considered "over-engineering" when conducting a review.  After all, if your job is review and improve architectures, it may not be over-engineering to you to recommend AWS Managed Kafka as a Service to introduce a message bus to decouple applications when simple SQS will work just fine.  With that in mind, the company suggested getting to know technical staff as part of the review.  By having an understanding of how the engineering staff likes to work, whether it's open source over managed solutions, Windows over Linux, Kubernetes or plain ECS; this information helps to shape recommendations.

## Need a workload reviewed?
If you like what you read, reach out!  We'd love to conduct a review of one of your workloads.  Check out our [Amazon Partner Network page](https://en.ippon.tech/aws-well-architected-reviews/) to begin scheduling a WAR.  Or, reach out to me directly at dferguson@ipponusa.com and I'll work with you to schedule your first review with our team of highly skilled reviewers.
