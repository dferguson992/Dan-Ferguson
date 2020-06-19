---
layout: posts
title: Snowflake External Functions
permalink: /posts/snowflake-external-functions
published: true
---

Snowflake is a cloud-based data warehousing company.  They specialize in provisioning on-demand compute and elastic storage for data warehousing applications on a pay-per-use pricing model.  [In recent years](https://www.analytics.today/blog/oracle-vs-snowflake) they've dominated the [cloud data-warehouse](https://www.prnewswire.com/news-releases/snowflake-more-than-triples-revenue-and-customer-base-doubles-post-money-valuation-all-in-just-one-year-300793857.html) industry.  Recently they announced a few new product features.  In this post, we're going to discuss one of those features, External Functions.
## External Functions
[External Functions](https://docs.snowflake.com/en/sql-reference/external-functions-introduction.html) in Snowflake are functions hosted on a different platform.  What does this mean?  Let's break it down.
[Functions](https://docs.snowflake.com/en/sql-reference/user-defined-functions.html) are pieces of Javascript or SQL code that execute in your data warehouse.  The code itself is written by developers and is intended to add features and functionality to your data warehouse, beyond the traditional SQL commands that Snowflake already offers.  External Functions are similar, except they are not hosted on Snowflake, and do not execute within the context of your data warehouse.  Instead, they are hosted elsewhere, accessed via an HTTP endpoint, and computed using third-party compute resources.  When making a call to a function, the value is computed using Snowflake's compute resources.  External Function values are computed using external resources, and the results are returned via HTTP.
For example, if you had access to an external service that mapped zip codes to cities, the typical approach to leveraging this service without external functions would be to import the keys and values of cities and zip-codes to a table and then query that table.  Then you could select cities from the `zipcode_to_city` table where the zip-code is some value.  For example,
1. Create the table `zipcode_to_city`.
2. Create an ETL pipeline that batch loads `zipcode_to_city` with the zipcodes and cities from your source system.
3. Automate this ETL pipeline to update the table when/if changes happen.
4. Run `select city from zipcode_to_city where zip = '11102';`.
5. (Optional) Create a function that replaces this query for you in other queries.
This is standard practice for integrating databases with external systems.  The problem here is you have to maintain an ETL pipeline to keep your downstream database up to date.  Additionally, you have another table to query which could impact the performance of queries downstream.  Now imagine having to do this for several external systems of record beyond just zip code - city mappings.  It's easy to see how this could get complicated quickly.
Now, with external functions, you can create the `zipcode_to_city_external_function` and get the same data as you would have if you imported the table.  The only difference now is that instead of importing a table to Snowflake, you import only the data you need, when you need it, via HTTPS.  After configuring your function (which we'll see how to do later), you can call it in SQL like this:
```sql
select zipcode_to_city_external_function(zipcode);
```
The function `zipcode_to_city_external_function` replaces the typical call to your reference table and the value is imported and used in the rest of your SQL statement.  Now you don't have to maintain a reference table of zip-code – city mappings because you've offloaded that functionality to a third-party service.  
## Creating an External Function
I'm not going to spend time breaking out syntax associated with creating an external function.  If you want to know the syntax options, check the Snowflake documentation associated with [External Functions](https://docs.snowflake.com/en/sql-reference/sql/create-external-function.html#usage-notes).  Instead, I'll go through the typical workflow required to create and use an External Function on AWS.
1. Configure access to external function (IAM)
2. Create an API Integration object in Snowflake (API Gateway)
3. Create the External Function
### Configuring Access 
The first thing to do is to create the IAM assets used by Snowflake to make calls to your AWS hosted function.  Due to limitations we'll explore later in this post, Snowflake must call API Gateway.  As a result, we'll create this policy where you'll specify your own hosting API ARN:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "VisualEditor0",
            "Effect": "Allow",
            "Action": [
                "apigateway:DELETE",
                "apigateway:PUT",
                "apigateway:PATCH",
                "apigateway:POST",
                "apigateway:GET"
            ],
            "Resource": "<API GATEWAY ARN>"
        }
    ]
}
```
Now we create a role.  This role is a little different from your typical role as it requires access from a third party.  Note in the image below we select "Another AWS account" as the trusted entity, specifying a dummy ID in the external ID field.  When creating this role, specify your own account's Account ID in the Account ID field.  After you've done this, add the policy we created above to this role and complete the role creation.
[!](https://blog.ippon.tech/content/images/2020/06/Screen-Shot-2020-06-15-at-10.29.03-AM.png)
### Configure Snowflake-Side Access
What we've done to this point is setup a templated access to AWS from Snowflake on the AWS side.  There's still some configuration we need to create on the Snowflake side before we can go back and finalize our AWS configuration.  
External Functions require a Snowflake construct called an [API Integration](https://docs.snowflake.com/en/sql-reference/sql/create-api-integration.html).  These constructs are logical endpoints within Snowflake that store references to external APIs.  See the below code snippet for more information.
```sql
CREATE API INTEGRATION IF NOT EXISTS external_integration
    API_PROVIDER = aws_api_gateway
    API_AWS_ROLE_ARN = '<ARN OF ROLE CREATED BEFORE>'
    ENABLED = TRUE
    API_ALLOWED_PREFIXES = ('<API GATEWAY DEPLOYMENT URL')
    COMMENT = 'AWS Hosted API Gateway Integration'
    ;
```
By running this snippet and replacing the values referenced with the role we created earlier and the API deployment that hosts your external function, we have created an API Integration.  This is important for configuring access for the external function later on.  
Once this stage is created, we need to gather some information about it.  Run the following command and be sure to save the output for your reference:
```sql
DESC API INTEGRATION external_integration;
```
The output for this command shows some meta-data about the API Integration that we specified when creating the integration.  It also includes two important fields called **API_AWS_EXTERNAL_ID** and **API_AWS_IAM_USER_ARN**.  We'll need these fields to continue configuring access  to AWS.  Record these field values and return to the AWS IAM console.
### Modifying AWS Access
Go back to AWS IAM and modify the role we created at the beginning of this post.  Navigate to the "Trust Relationship" tab and edit this policy.  When you open the policy document, it should look something like this:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "<YOUR ACCOUNBT ID>"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "<DUMMY ID>"
        }
      }
    }
  ]
}
```
Replace the ARN containing your Account ID with the **API_AWS_IAM_USER_ARN** value from the previous query we ran in Snowflake.  Then replace the Dummy ID with the **API_AWS_EXTERNAL_ID** from the previous query.  The results should look like the following:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "AWS": "<API_AWS_IAM_USER_ARN>"
      },
      "Action": "sts:AssumeRole",
      "Condition": {
        "StringEquals": {
          "sts:ExternalId": "<API_AWS_EXTERNAL_ID>"
        }
      }
    }
  ]
}
```
What this does is allow the Snowflake asset you created with the **API_AWS_EXTERNAL_ID** and **API_AWS_IAM_USER_ARN** specified to assume the role with the permissions we gave it.  Based on this blog post, that means access to call an API Gateway endpoint.  If you modify this role to have new policies though, that means you will grant this third party service all of those new policies.  For more details,  check out this how-to guide published by Snowflake on this topic.
### Creating the External Function
After all that back and forth, this is the easy part.  Following the [syntax for an External Function](https://docs.snowflake.com/en/sql-reference/sql/create-external-function.html#usage-notes), we should be sending something like the following to Snowflake:
```sql
CREATE OR REPLACE SECURE EXTERNAL FUNCTION zipcode_to_city_external_function (zipcode VARCHAR)
  RETURNS VARCHAR
  NOT NULL
  RETURNS NULL ON NULL INPUT
  VOLATILE
  COMMENT = 'Returns a city for the given zip-code.'
  API_INTEGRATION = external_integration
  HEADERS = ('content-type' = 'application/json')
  AS '<API ENDPOINT URL>'
  ;
```
Notice how we specify an empty list of parameters, a return type, and the API Integration object we created earlier.  The API Integration object allows this external function to assume the role we specified, giving Snowflake the ability to query API Gateway as an external service.
To use this external function, all we have to do is call our function like you would any User-Defined Function:
```sql
SELECT zipcode_to_city_external_function("11102");
```
The results of this function call will vary based on the back-end implementation, but in my demo the result is:
```csv
ZIPCODE_TO_CITY_EXTERNAL_FUNCTION("11102")
ASTORIA
```
The following is the backend implementation for this endpoint:
```nodejs
exports.handler = async (event) => {

    // MAGIC HAPPENS HERE
    city = CALL_TO_CITY_ZIP_MAPPER(event)
    const response = {
        data: [
            [0, city]
        ]
    };
    return response;
};
```
Obviously this is a hard-coded example.  I've hard-coded the response here to show the formatting of a response that Snowflake expects.  If you do not format your output like this you'll get an error like `external function top-level JSON object must contain "data" JSON array element`.   This may seem strange at first, but it makes a lot of sense when you break it down.  Snowflake is, at the end of the day, a database.  It stores data in a structured way; when it integrates with third parties it expects the third party to respect that data quality requirement.  As a result, we have this strange JSON-esque syntax being returned by the lambda function.  For more information on this, check out Snowflake's documentation about [creating External Functions for different platforms](https://docs.snowflake.com/en/sql-reference/external-functions-general.html#data-format-sent-from-snowflake).
##  Limitations & Gotcha's to External Functions
* External Functions need to be called through an HTTP proxy service.  This service is referenced in the API_INTEGRATION field.  Currently, the only supported API Integration objects are those hosted on API Gateway.
* External Function calls cannot be used to define the default value of a column when creating a table.
* External Functions must return scalar values that match the return type of the external function.  Returned values must also match the number of rows which are being used.
* External Functions always make POST requests.
* The top-level response must be a series of name-value pairs sent as a JSON array labeled "data."
* Lambda functions that integrate with API Gateway by proxy will have to include the expected API Gateway response object, as well as the expected Snowflake data object.
* Backend integrations do not have to be Lambda functions, they just have to be hosted via API Gateway (at this time).
* If you ever re-create your API Integration object in Snowflake, you will have to modify the IAM role.  You just have to modify the API_AWS_EXTERNAL_ID, the API_AWS_IAM_USER_ARN field should stay the same (unless your Snowflake user changes). 
For more details on the limitations of external functions, and the additional considerations you should take when creating external functions, check the [Snowflake documentation](https://docs.snowflake.com/en/sql-reference/external-functions-introduction.html#limitations-of-external-functions) on the subject.
