---
layout: posts
title: Comparing Java LTS Releases
permalink: /posts/comparing-java-lts-releases
published: false
---

Hey ya’ll.  Got an update on Contracker.

I’ve been toying around with the Aurora Serverless instances and I’ve got a lesson-learned to share.

Aurora MySQL 5.6 is the only flavor of serverless Aurora.  You can’t have a newer version of MySQL than 5.6.  Is this inherently an issue?  No.  There’s nothing wrong with using version 5.6 of MySQL to allow for an internal application to utilize a serverless instance of RDS.  Is this an issue for us?  It could be, here’s why.  

When creating a new user on this database (contracker-admin) to write to the tables associated with the contracker app, you must set the “log_bin_trust_function_creators” parameter to 1.  To do this, you need to create a new parameter group in RDS.  RDS uses parameter groups to manage common parameter settings across databases.  For Aurora 5.6, there is a default parameter group already in existence called default.aurora5.6 and it cannot be modified.  This is because you need to create a custom parameter group if you want to modify default settings.  The issue is, when you create a default parameter group for Aurora MySql, the only version of MySql you can create the parameter group for is 5.7.  So if you create the parameter group for 5.7, you cannot apply it to your serverless instance of Aurora MySQL 5.6.

Okay, no big deal, we can just copy the parameter group!  There’s an AWS CLI command for this:
aws rds copy-db-parameter-group --source-db-parameter-group-identifier default.aurora5.6 --target-db-parameter-group-identifier contracker.aurora5.6 --target-db-parameter-group-description "Parameter Group for Contracker DB" --r "us-east-1" --profile "ippon-sts"

When you run this command, you get:
An error occurred (InvalidParameterValue) when calling the CopyDBParameterGroup operation: The parameter DBParameterGroupName is not a valid identifier. Identifiers must begin with a letter; must contain only ASCII letters, digits, and hyphens; and must not end with a hyphen or contain two consecutive hyphens.

A quick google search shows this issue is systemic.  You cannot create list a parameter group containing periods in the name, so you cannot copy a default parameter group from the CLI or the console.  You have to create a parameter group manually, sourcing a “parameter group family” which is different than the default parameter groups.  These “parameter group families” represent a subset of the supported database flavors on RDS.  Guess which flavor is not part of this family: Aurora-MySql 5.6.  The reasoning here is simple, you cannot use Aurora MySql 5.6 serverless RDS instances for anything beyond the exploratory database deployment.

So where does that leave us now?  Well, we have four options:
1. Use the default admin user for everything and maintain the serverless instance of Aurora MySql 5.6.
2. Upgrade to a different RDS flavor like Aurora MySql 5.7, MySql 8.0, Postgres etc. and loose the serverless capabilities.
3. Move to NoSQL.

My two cents on the above: one is out because it’s just bad practice and if this is the issue we see on Aurora MySQL 5.6 now before we’ve put any data in it, we’re gonna give ourselves headaches tomorrow by sticking with serverless RDS today.  Two is a good option, we’ll still have a true SQL database and I think that was a learning experience we were all keen on during last night’s call.  Three is probably the easiest option and the most extensible.  

I personally recommend we start with DynamoDB and migrate to RDS at a later date.  I think RDS is a great engine for learning about enterprise database management, but figuring all of that out now will be biting off more than we can chew.  I’ve removed the contracker infrastructure in RDS, but kept the DDL saved query when we come back to this later on down the line.  Additionally, I’ve setup a Cloud9 development environment for us to use: contracker-dev-env during Monday’s session.  I encourage everyone to take a look if you plan to attend.


The plan at this point is to use DynamoDB to start our development work, with the intent to migrate to RDS in the future.  I realize I’ve made this decision in something of a vacuum last night, so if you have alternative suggestions or you know of a work around to the issue I’ve described above I’d love to hear it!  I encourage everyone to test this out for themselves, I may have missed something last night while exploring this; but it’s also a great learning experience for those interested!


Please comment your thoughts even if you agree I don’t want to make decisions in a vacuum as this is a collaborative project.  
