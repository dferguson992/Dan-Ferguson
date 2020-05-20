---
layout: posts
title: Building a HIPAA Compliant Message Bus with Solace
permalink: /posts/hipaa-compliant-message-bus
published: true
---
## What is Solace?
Solace are the makers of the Solace PubSub+ Platform which is used to event enable your enterprise. The core of the platform is the Solace PubSub+ Advanced Event Broker that comes in hardware, software, and messaging as a service in your favorite public cloud.

## Cloud-Native Solace
In the last few years, Solace has begun investing heavily into cloud based messaging solutions. Using tools like the Solace Cloud Console, you can quickly provision redundant and secure virtual solace appliances in any cloud you choose. But Solace messaging isn't just fast, it is reliable, extensible, and secure. Solace virtual routers can seamlessly be deployed as an HA pair in a single region and DR can be setup across regions to ensure availability constraints that your enterprise demands. Solace routers support nearly every major messaging protocol, the list of protocols not supported is shorter than list of protocols that are. Solace routers are also secure. Administrative access for a private cloud deployment is as secure as the EC2 instances in your Amazon console.
![](https://blog.ippon.tech/content/images/2020/02/Solace-PubSub-Platform-Diagram-1.png)
All of these built-in features, coupled with native cloud deployments with a very generous free-tier, make Solace an obvious option for businesses outside the financial space.  Companies spend thousands of dollars and person-hours building reliable messaging deployments that are secure and cost-effective.  Now you can have those features at the click of a button thanks to the cloud.  (Check out my last article talking about exactly how easy it is to do this.)  But not every business vertical is convinced cloud-based solutions like Solace fit their needs.  One of those  business verticals is healthcare and insurance.  The real question for these verticals is around HIPAA.  In this article we'll discuss what HIPAA is and why Solace is absolutely a viable choice for messaging in a HIPAA compliant environment.  By taking the time to properly configure your PubSub+ services, achieving HIPAA compliance with Solace is quite straightforward. I'll show you how to get started in this article.

## What is HIPAA?
HIPAA, or Health Insurance Portability and Accountability Act, is a piece of legislation passed in 1996 that puts regulation around the security of Personal Health Information (PHI).  Health insurance organizations deal with PHI all the time, this is the very nature of an insurance claim.  In the 24 years since this legislation was made law, the digital world has changed drastically, but compliance requirements remain the same.  For a full list of HIPAA compliance requirements, check out this website.  The list of requirements are extensive, but they do boil down to the following short-list of absolute necessities.
1. Implement a means of access control. This not only means assigning a centrally-controlled unique username and PIN code for each user, but also establishing procedures to govern the release or disclosure of ePHI during an emergency.
2. Introduce activity logs and audit controls. The audit controls required under the technical safeguards are there to register attempted access to ePHI and record what is done with that data once it has been accessed
3. Enforce policies for the use/positioning of workstations.  Policies must be devised and implemented to restrict the use of workstations that have access to ePHI, to specify the protective surrounding of a workstation and govern how functions are to be performed on the workstations.
4. Enforce policies and procedures for mobile devices. If users are allowed to access ePHI from their mobile devices, policies must be devised and implemented to govern how ePHI is removed from the devices if the user leaves the organization or the device is re-used, sold, etc.
5. Conduct regular risk assessments. Among the Security Officer´s main tasks is the compilation of a risk assessment to identify every area in which ePHI is being used, and to determine all of the ways in which breaches of ePHI could occur.
6. Introducing a risk management policy. The risk assessment must be repeated at regular intervals with measures introduced to reduce the risks to an appropriate level. A sanctions policy for employees who fail to comply with HIPAA regulations must also be introduced.
7. Develop a contingency plan. In the event of an emergency, a contingency plan must be ready to enable the continuation of critical business processes while protecting the integrity of ePHI while an organization operates in emergency mode.
8. Restrict third-party access. It is vital to ensure ePHI is not accessed by unauthorized parent organizations and subcontractors, and that Business Associate Agreements are signed with business partners who will have access to ePHI.

## Assessment Assumptions
This article is about defining a HIPAA compliant Solace implementation for companies dealing with PHI.  Let's define what a typical environment would look like for a company like this.
* User login to applications is handled by a central authentication/authorization platform like LDAP or Active Directory.
* Applications all have their own usernames for the various integrations they possess, but end user activity is still logged and tracked for audit and control purposes.
* Datastores, third party integrations, and application integrations have a application-user level permissions.  A typical example would be grants to select data from a table in a database for a specific application.  
* The company bound to HIPAA compliance is presumed to have a secure network from within the physical premises of the company.  Remote logins are secured through IPSec, VPNs, or some other secure communication protocol.
* Applications used by the organization are already HIPAA compliant.
* The organization already has a HIPAA compliant cloud-adoption policy in place that honors the phyical requirements of a HIPAA compliant workspace.
* The organization already has a HIPAA Risk Assessment & Management Policy in place that can be applied to new services like PubSub+.

This list of 5 assumptions about an organization trying to be HIPAA compliant is fairly general.  Federated user login, application activity logging, intelligent access controls around data, and secure networking are all standard requirements for any business that makes money from data.  PHI data and Insure-tech companies are no exception.  Given the above mentioned general settings for an organization, we will discuss how the required aspects of HIPAA can be honored in a Solace Cloud deployment.
* TLS / encryption to secure PHI/PII data in transit)
* Encrypted storage to secure PHI/PII data at rest
* Fine-grained access controls to govern and ensure only authorized systems can publish data and can receive data
* Auditing of changes to security configurations

## Technical Safeguards
### Implement a Means of Access Control
Access control in Solace is a built-in, out of the box component of Solace.  Access control on Solace are baked in down to the underlying operating system.  For cloud deployments however, the true HIPAA compliant access control comes from User Authentication/Authorization, ACLs, and Client-Profiles.
Authenticating against a Solace appliance can be done a number of ways.  The simplest is with a username and password.  This is the default setting for free-tier deployments.  There are a number of other built-in authentication solutions used by Solace that are quite extensible; an entire series of blogs could be written on this topic alone.  Concerning HIPAA compliance, Solace integrates easily with LDAP.  By authorizing against LDAP, your Solace access controls have the same HIPAA compliance as your LDAP configuration.  Configuring LDAP to be HIPAA compliant is a well-established and well-documented process; it suffices to say that if your LDAP configuration is HIPAA compliant, your Solace configuration is HIPAA compliant.
In addition to LDAP integration, Solace has some built-in configuration options that can supply fine-grained access control down to the topic level.  Like most messaging systems, Solace has Access-Control Lists.  ACLs in Solace define the topics a user can publish and subscribe to, without the user having to know ahead of time.  
![](https://blog.ippon.tech/content/images/2020/02/Screen-Shot-2020-02-28-at-2.17.09-PM.png)
![](https://blog.ippon.tech/content/images/2020/02/Screen-Shot-2020-02-28-at-2.17.53-PM.png)
If the user attempts to pub/sub against a topic not on their ACL, an error will be thrown.  Additionally, Solace ACLs can enforce IP whitelists or blacklists.  This means a user is only allowed to pub/sub messages from a very specific network location.
![](https://blog.ippon.tech/content/images/2020/02/Screen-Shot-2020-02-28-at-2.14.58-PM-1.png)
Typically, this is a physical location in an office or the end of an encrypted VPN tunnel.  This kind of control is not required by HIPAA, but it is a recommended step to take when trying to become HIPAA compliant.
Another great feature of Solace that provides fine-grained, HIPAA compliant access control are client profiles.  Like the various methods of authentication to Solace, client profiles are an extensive topic in Solace configuration that require an in-depth knowledge of low-latency networking to fully grasp.  Typically, these settings are worth modifying as a workaround for upstream networking issues.  However, it's worth noting some of the less specific features that could be configured in a HIPAA compliant environment.
![](https://blog.ippon.tech/content/images/2020/02/Screen-Shot-2020-02-28-at-2.23.07-PM.png)
When configuring a client profile, it is important to remember that clients using this profile will be restricted to the actions and settings in that profile.  There are some client specific settings that can configured, but general usability is restricted at the profile level.  For example, if you never want a client to connect to a queue, you would not allow them to receive guaranteed messages.  If you wanted to specify a service level user that could only create bridged connections (connections between different Solace Message VPNs or Services), you would enable "Connect as a Bridge."  If you only want to publish messages when there is an active subscriber receiving them, you would enable the "Reject Messages to Sender on No Subscription Match Discard" setting.  These are all standard, high-level settings that define the behavior for a set of clients using your Solace device.  HIPAA compliance comes in with the "Downgrade Connection to Plain Text" setting.  This is a setting you would immediately disable when configuring Solace.  Under no circumstances should you downgrade the encryption on a connection in a HIPAA compliant environment.  By disabling this setting, all users on that client profile will automatically follow this setting.

### Introduce Activity Logs and Audit Controls
If you want to view logs on a Solace router, simply subscribe to the topic #LOG/>.  This does require a small amount of configuration on the router side (see below), but once enabled, your configured log messages will be routed to the appropriate topic for consumption.  
![](https://blog.ippon.tech/content/images/2020/03/Screen-Shot-2020-03-17-at-12.35.28-PM.png)
Instead of writing to log files, Solace will track activity by publishing messages to the internal logging topic #LOG.  By setting up a queue or an always-on consumer bound to the #LOG/> topic, you will always have up to date logs for your app.  Furthermore, log messages are broken out by topic hierarchy.  This allows you to have very granular logging configurations across all of your consumers.  In short, Solace's logging capabilities extends to meet your organization's HIPAA requirements; you just may have to think outside of the box about acquiring those logs.  You no longer need to setup an FTP site or run a `tail -F logs/*`.  If you think about logs as messages, it becomes very easy to acquire logs in a scalable manner to achieve HIPAA compliance.

## Administrative Safeguards
### Developing a Contingency Plan
Solace PubSub+ appliances come with the pre-configured option of establishing highly available routing, with the option to add a disaster recovery node.  This triplet of appliances ensures always-on performance with no data loss in case of a disaster.
### Restricting Third-Party Access
The rules governing third-party access to a PubSub+ service are as straightforward as restricting access to corporate e-mail.  Secure passwords and proper network configuration make PubSub+ services quite secure from third-party access.  This is true for any cloud deployment.  Consider AWS's VPC construct.  By default, it secures resources access through security groups, route tables and network access control lists.  When deploying a Solace PubSub+ appliance inside a corporate VPC, your PubSub+ appliance is bound to the same access restrictions which protect all of your assets from third-party access.

## Final Thoughts on HIPAA Compliance with Solace
Solace is a HIPAA compliant messaging solution.  Solace appliances have always had this capability, even before the company released a cloud-native deployment of their messaging technology.  By taking the time to properly configure your PubSub+ services, achieving HIPAA compliance is quite straightforward.  Here's a short-list of recommendations to get your organization started.  From here, use your organization's specific requirements to ensure PubSub+ services will assist you in all your messaging needs.

* Define a logging consumer that writes #LOG/> messages to Splunk, or an Elasticsearch cluster.
* Deploy your PubSub+ in an AWS VPC within a private subnet.  Allow routing between a primary and secondary appliance, with fail-over traffic going to the DR appliance in a different region.
* Deploy a PubSub+ image or Virtual Machine into your private cloud and manage it completely independent of Solace and the Cloud Console.  (AWS has an AMI on the Marketplace that allows you to deploy a Solace PubSub+ appliance using the EC2 service for example.)
* Impart failover best-practices into your environment.  In a triplicate deployment, the requirements are a little tricky.  Check out [this document from Solace](https://docs.solace.com/Configuring-and-Managing/Redundancy.htm) that defines best practices for failover in a PubSub+ environment.
* Establish a white-list of subnets that are permitted to connect to the appliances.  From there, determine which clients belong to which subnets and allocate topics they are permitted to publish and subscribe to using ACLs.
* Apply a security layer to the Solace APIs that meets your organization's security standards.  For example, this layer could publish all messages using your organization's encryption policies.  

For more information on Solace's PubSub+ service, or if you have a specific question, check out https://solace.community/.  This website is constantly monitored by Solace professionals looking to answer any question you may have.
