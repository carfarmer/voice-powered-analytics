# Workshop Overview
In this workshop you will build a voice powered analytic engine that you can take back to your stakeholders to deliver valuable company insights.   Common questions that may be asked, “Alexa, how many Unique Users did our site have last month?” and “Alexa, how many orders have breached their delivery SLA this week?”.

## The primary products used in this workshop are:
All workshop attendes will need an AWS account with access to the following products. AWS will **NOT** be providing temporary accounts for this workshop. AWS will be providing a **TODO** $25 credit to cover the expense of the workshop. 

### Pre-workshop Checklist
Please make sure you have the following availabile prior to the workshop.

- [ ] AWS Account with root access
- [ ] Ability to create new IAM policies and roles
- [ ] Full access to Athena – Clusterless Query Engine
- [ ] Full access to Quicksight – Interactive BI Visualizations
- [ ] Full access to S3 – Limitless and durable object store
- [ ] Full access to Lambda – Event-triggered functions
- [ ] Full access to DynamoDB – Managed NoSQL database
- [ ] Full access to Alexa – Voice-powered skills

**Note** There are two steps that differ from the typical AWS workflow. 

* Development of an Alexa skill requires creation of an account at [Amazon Developer](https://developer.amazon.com/alexa-skills-kit) If you have not created an account yet, please do so before the workshop.
* Using QuickSight requires signing up for the service on a per user basis. Please conplete this step before the workshop to save on time.  

## Workshop Steps

This workshop is designed for first time users of Athena and Alexa. We have broken the workshop into three Steps or focus topics. These are:

* **Alexa Skill Building Step**
* **BI and Data Discovery Step**
* **Advanced Alexa Skill Building Step**

We expect most attendes to be able to complete both the Alexa Skill Building and BI and Data Discovery Steps and if time permits or if you are as excited about Alexa Notifications as we are you can focus in on the optional path, Advanced Alexa Skill Building.

We have provided cloud formation templates and solutions for all steps where the attende is expected to write code. Generally these come in two flavors:

1. Full solution where the attende does not have to write any code
1. Partial solution where the attende can author key selections of the code and double check thier work. This path is the recomended path as it provides for the most learning. If time becomes an issue, attendes will always have access to the full solitions so be bold!

In addition to deciding bewteen using the full or partial solutions. The attende can also choose to focus in on the Big Data portion or the Voice powered portion. Please spend the time in the workshop you find most interesting and use our full solutions for anything you have already mastered or are not interested in. The partial solution is designed to give you a head start, but still require key additions from the attende. 


## BI and Data Discovery Step

We will be using a dataset created from Twitter data related to AWS re:Invent 2017. In short, tweets with the #reinvent hashtag or to/from @awsreinvent 

This dataset is avilable as:

```
US-EAST-1 
s3://aws-vpa-tweets/
```

### Step 1 - Create an Athena table

There is no need to copy the dataset to a new bucket for the workshop. The data is publicly available. You will however need to create an Athena database and table to query the data. The twitter data is stored in s3 in a JSON format which Athena has native support for. 

<details>
<summary><strong>Full solution - Athena table (expand for details)</strong></summary><p>

1. In your AWS account please go to Athena query editor.
1. You need to create a new table using this for the external table DDL:

```SQL
CREATE EXTERNAL TABLE tweets(
  id bigint COMMENT 'Tweet ID', 
  text string COMMENT 'Tweet text', 
  created timestamp COMMENT 'Tweet create timestamp', 
  screen_name string COMMENT 'Tweet screen_name',
  screen_name_followers_count int COMMENT 'Tweet screen_name follower count',
  place string COMMENT 'Location full name',
  country string COMMENT 'Location country',
  retweet_count int COMMENT 'Retweet count', 
  favorite_count int COMMENT 'Favorite count')
ROW FORMAT SERDE 
  'org.openx.data.jsonserde.JsonSerDe' 
WITH SERDEPROPERTIES ( 
  'paths'='id,text,created,screen_name,screen_name_followers_count,place_fullname,country,retweet_count,favorite_count') 
STORED AS INPUTFORMAT 
  'org.apache.hadoop.mapred.TextInputFormat' 
OUTPUTFORMAT 
  'org.apache.hadoop.hive.ql.io.HiveIgnoreKeyTextOutputFormat'
LOCATION
  's3://aws-vpa-tweets/tweets/'
```
1. Test the Athena table with a simple `SELECT` statement:

```SQL
SELECT COUNT(*) AS TOTAL_TWEETS FROM tweets;
```

</p></details>

<details>
<summary><strong>Partial solution - Athena table (expand for details)</strong></summary><p>

TODO: Use AWS Glue to discover and build a DDL.

</p></details>

### Step 2 - Explore the data using Quicksight

1. Launch the QuickSight portal and in the upper right choose **Manage data**
1. Now in the upper left choose **New data set**
1. You will see tiles for each of the QuickSight supported data sources. From this page select the **Athena** tile. 
1. When asked for the dataset name you can choose anything you like, for our example we use **tweets-dataset** You can choose to validate that SSL will be used for data between Athena and QuickSight. Finish be selecting **Create data source**
1. Now we need to choose the Athena table we created in **Step 1**. For our example we used the **Default** database, with a table name of **tweets**. Finish by clicking on **Select**. 
1. You will now be asked if you want to use spice or direct query access. For our use case lets select **SPICE**. Click **Visualize** when done. 
1. QuickSight will now import the data into SPICE. Wait until you see **Import Complete**. Then close the summary window. 
1. Add the **created** field from the Athena table by dragging it from the Field list to the empty AutoGraph window.
1. From the Visual types, select **Vertical bar chart**
1. Add another Visual by selecting the **"+ Add" and then Add visual**
1. On this new graph, lets add the **country** field. 
1. As you can see, lots of tweets do not include which country the tweet was created in. Lets filter these results out. Click on the large bar labeled **none**, then select **exclude** from the pop up window. As you can see the tweets without a location were excluded.
1. Lets change the visual from a bar chart to a pie chart. Select the entire visual, then from the bottom right select the **pie chart** visual.


### Step 3 - Create a query to find the number of reinvent tweets 


</p></details>

<details>
<summary><strong>Full solution - Athena Query (expand for details)</strong></summary><p>

1. We need to produce an integer for our Alexa skill. To do that we need to create a query that will return our desired count.
1. Goto the Athena AWS Console page. From there select the **Default** Database
1. Click the new query button. The Query text to find the number of #reinvent tweets is:  `SELECT COUNT(*) FROM tweets`
1. Once you are happy with the value returned by your query you can move to **Step 4**, otherwise you can experiment with other query types. A few examples are listed below.

```SQL
--Total number of tweets
SELECT COUNT(*) FROM tweets

--Total number of tweets in last 3 hours
SELECT COUNT(*) FROM tweets WHERE created > now() - interval '3' hour

--Total number of tweets by user chadneal
SELECT COUNT(*) FROM tweets WHERE screen_name LIKE '%chadneal%'

--Total number of tweets that mention AWSreInvent
SELECT COUNT(*) FROM tweets WHERE text LIKE '%AWSreInvent%'

```

</p></details>

</p></details>

<details>
<summary><strong>Partial solution - Athena Query (expand for details)</strong></summary><p>

**Note:** If you would like example query strings, please review this steps Full solution.

1. We need to produce an integer for our Alexa skill. To do that we need to create a query that will return our desired count.
1. Athena is widely compatable with Presto. You can learn more about it from our [AWS Athena Getting Started](http://docs.aws.amazon.com/athena/latest/ug/getting-started.html) and the [Preto Docs](https://prestodb.io/docs/current/) web sites
1. You can query whatever you like as this value will be used later from our Alexa skill


</p></details>

### Step 4 - Create a lambda to query Athena

In this step we will create a **Lambda function** that runs every 5 minutes. The lambda code is provided but please take the time to review the function.

#### - Create the lambda to query Athena
1. Go to the [AWS Lambda console page](https://console.aws.amazon.com/lambda/home?region=us-east-1#/functions)
1. Click **Create Function** 
1. We will skip using a blueprint to get started and autor one from scratch. Click **Author one from scratch** 
1. Leave the trigger blank for now. Click **Next** without adding a trigger from the Configure triggers page.
1. Give your Lambda function a unique name. For example you can use **Athena_Poller** for the query name. For runtime select **Python 3.6**
1. Select inline code and then use the:

TODO: Clean up the Poller code

```Python
import boto3
import csv
import time
import os
from urllib.parse import urlparse


def lambda_handler(event, context):
    query = os.environ['query']
    result = run_athena_query(query, os.environ['database'], os.environ['s3_output_location'])
    upsert_into_DDB(os.environ['Metric_Name'], result, context)
    return {'message': "{0} reinvent tweets so far!".format(result)}


# runs athena query, open results file at specific s3 location and returns result
def run_athena_query(query, database, s3_output_location):
    athena_client = boto3.client('athena', region_name=os.environ['region'])
    s3_client = boto3.client('s3', region_name=os.environ['region'])
    queryrunning = 0

    #  kickoff the Athena query
    response = athena_client.start_query_execution(
        QueryString=query,
        QueryExecutionContext={
            'Database': database
        },
        ResultConfiguration={
            'OutputLocation': s3_output_location
        }
    )

    # Log the query execution id
    print('Execution ID: ' + response['QueryExecutionId'])

    # wait for query to finish.
    while (queryrunning == 0):
        time.sleep(2)
        status = athena_client.get_query_execution(QueryExecutionId=response['QueryExecutionId'])
        results_file = status["QueryExecution"]["ResultConfiguration"]["OutputLocation"]
        if (status["QueryExecution"]["Status"]["State"] != "RUNNING"):
            queryrunning = 1

    # parse the s3 URL and find the bucket name and key name
    s3url = urlparse(results_file)
    s3_bucket = s3url.netloc
    s3_key = s3url.path
    print(s3_key)

    # download the result from s3
    s3_client.download_file(s3_bucket, s3_key[1:], "/tmp/results.csv")

    # parse file and update the data to DynamoDB
    with open("/tmp/results.csv", newline='') as f:
        reader = csv.DictReader(f)
        for row in reader:
            print(row[os.environ['resultCol']])
            return row[os.environ['resultCol']]  # return result


def upsert_into_DDB(nm, value, context):
    region = os.environ['region']
    dynamodb = boto3.resource('dynamodb', region_name=region)
    table = dynamodb.Table(os.environ['DDB_Table'])
    try:
        response = table.put_item(
            Item={
                'metric': nm,
                'value': value
            }
        )
        return 0
    except Exception:
        print(str(e))
        return 1
```

1. Add the following for environment variables TODO add env, IAM Role
1. Set the following Environment variables: TODO: Remove spaces in table names TODO: Create bucket for Athena results

```
database = tweets
DDB_Table = VPA_Metrics_Table
Metric_Name = Reinvent Twitter Sentiment
query = SELECT count(*) FROM default."tweets"
resultCol = _col0
region = us-east-1
s3_output_location = s3://aws-vpa-athena-query-results/poller/

```

1. From the **Lambda function handler and role** ensure the Handler is set to `lambda_function.lambda_handler` and the Existing role to `lambda_athena_poller`
1. Select Adnanced Settings in order to configure the Timeout value to **1 minute**
1. Click **Next**
1. From the review page, select **Create Function**


#### - Create a CloudWatch Event Rule to trigger Lambda

1. Go to the [CloudWatch Events Rules console page](https://console.aws.amazon.com/cloudwatch/home?region=us-east-1#rules:). 
1. Click **create rule**
1. From the create rule page in the Event Source section. Select **Schedule** followed by **fixed rate** with a value of **5** minutes.
1. From the Target section select **Add target**, then **lambda function**, followed by the new query we just created, **Athena_poller**.
1. Next click on the **Configure Details**
1. Give your rule a name, in this case **every-5-min**
1. Unselect the **Enabled** button to disable the trigger and then select **Create rule** 

#### - Create an IAM Role for the Athena poller Lambda

1. Go to the [IAM Roles Console Page](https://console.aws.amazon.com/iam/home?region=us-east-1#/roles) 
1. Click on **Create Role** Button to create a new IAM Role
1. Make sure the **AWS Service** and **Lambda** are selected for the Role Type and click **Next: Permissions**.
1. Click on the **Create policy** button, followed by **Create your own policy**, then name the new policy **lambda_athena_poller** use the IAM Policy Document below

```JSON
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1506907773887",
      "Action": "cloudwatch:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506907792079",
      "Action": "logs:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506907804049",
      "Action": "athena:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506907824634",
      "Action": "xray:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506907846668",
      "Action": "s3:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506907885059",
      "Action": "dynamodb:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Stmt1506908036515",
      "Action": "glue:*",
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```

1. You select the new policy you created for this roles permissions. You can use the filter to search for **poller**. Now select **Next: Review** to review our role. 
1. Set the Role name to **poller_full_access** and click **create role**


## Alexa Skill Building Step

### Setting up Your Voice User Interface

There are two parts to an Alexa skill. The first part is the Voice User Interface (VUI). This is where we define how we will handle a user's voice input, and which code should be executed when specific commands are uttered. The second part is the actual code logic for our skill, and we will handle that in the next step of this step-by-step guide.

<details>
<summary><strong>Full solution - Setting up VUI (expand for details)</strong></summary><p>
  1. Go to the <a href="http://developer.amazon.com/">Amazon Developer Portal</a>. In the top-right corner of the screen, click the <b>"Sign In"</b> button. <br>(If you don't already have an account, you will be able to create a new one for free.)
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_1.png?raw=true">
  2. Once you have signed in, click the <b>Alexa button</b> at the top of the screen.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_2.png?raw=true">
  3.  On the Alexa page, choose the <b>"Get Started"</b> button for the Alexa Skills Kit.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_3.png?raw=true"><br>
  4.  Select <b>"Add A New Skill."</b> This will get you to the first page of your new Alexa skill.
  5.  Fill out the <b>Skill Information screen</b>. Make sure to review the tips we provide below the screenshot.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_4.png?raw=true"><br>
  <details>
 <summary><strong>Skill Information Tips (expand for details)</strong></summary><p> 

#### Skill Information Tips<br>
i.  <b>Skill Type</b> For this skill, we are creating a skill using the Custom Interaction Model. This is the default choice.
ii. <b>Language</b> Choose the first language you want to support. You can add additional languages in the future, but we need to start with one. (This guide is using U.S. English to start.)
iii.  <b>Name</b> This is the name that will be shown in the Alexa Skills Store, and the name your users will refer to.
iv. <b>Invocation Name</b> This is the name that your users will need to say to start your skill. We have provided some common issues developers encounter in the list below, but you should also review the entire <a href="https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/choosing-the-invocation-name-for-an-alexa-skill">Invocation Name Requirements</a>.
<table>
<thead>
<tr>
<th>Invocation Name Requirements</th>
<th>Examples of incorrect invocation names</th>
</tr>
</thead>
<tbody>
<tr>
<td>The skill invocation name must not infringe upon the intellectual property rights of an entity or person.</td>
<td>korean air; septa check</td>
</tr>
<tr>
<td>Invocation names should be more than one word (unless it is a brand or intellectual property), and must not be a name or place</td>
<td>horoscope; trivia; guide; new york</td>
</tr>
<tr>
<td>Two word invocation names are not allowed when one of the words is a definite article, indefinite article, or a preposition</td>
<td>any poet; the bookie; the fool</td>
</tr>
<tr>
<td>The invocation name must not contain any of the Alexa skill launch phrases and connecting words.  Launch phrase examples include "launch," "ask," "tell," "load," and "begin."  Connecting word examples include "to," "from," "by," "if," "and," "whether."</td>
<td>trivia game for star wars; better with bacon</td>
</tr>
<tr>
<td>The invocation name must not contain the wake words "Alexa," "Amazon," "Echo," or the words "skill" or "app."</td>
<td>hackster initial skill; word skills</td>
</tr>
<tr>
<td>The invocation name must be written in each language you choose to support.  For example, the German version of your skill must have an invocation name written in German, while the English (US) version must have an invocation name written in English.</td>
<td>kitchen stories (German skill)</td>
</tr></tbody></table>
 </p></details>
6.  Click the Next button to move to the <b>Interaction Model</b>.
7. Click on the <b>Launch Skill Builder (Beta)</b> button . This will launch the new Skill Builder Dashboard.
<IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_5.png?raw=true"><br>
8.  Click on the "<b>Code Editor"</b> item under Dashboard on the top left side of the skill builder.
9.  In the textfield provided, replace any existing code with the code provided in the <a href="https://github.com/voicehacks/setup-local-recommendations/blob/master/speech-assets/InteractionModel.json">Interaction Model</a>, then click "Apply Changes" or "Save Model".
10.  Click on the <b>"Dashboard"</b> button.
11.  Add some more sample utterances for your newly generated intents. These are the things a user would say to make a specific intent happen. Here are a few examples:
  - Show me my metrics / List my metrics
  - What is my {metric}
<IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_6.png?raw=true"><br>
12.  Click <b>"Build Model"</b> and <b>"Save"</b>
<IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_7.png?raw=true"><br>
13. If your interaction model builds successfully, click on <b>Configuration button</b> to move on to Configuration. In our next step of this guide, we will be creating our Lambda function in the AWS developer console, but keep this browser tab open, because we will be returning here on <a href="https://github.com/voicehacks/setup-local-recommendations/blob/master/step-by-step/3-connect-vui-to-code.md">Page #3: Connect VUI to Code</a>. 
<IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_8.png?raw=true"><br>
<br>If you get an error from your interaction model, check through this list:
   - Did you copy & paste the provided code into the appropriate boxes?
   - Did you accidentally add any characters to the Interaction Model or Sample Utterances?
</details>

### Configure Alexa Backend
Now that we've configured the voice interaction, let's set up our Lambda function to leverage your DynamoDB metrics and be triggered by the Alexa Skills Kit. 
<details>
<summary><strong>Full solution - Setting up Alexa Backend (expand for details)</strong></summary><p>
  1. Check your <b>AWS region</b>. For the reinvent workshop, we'll be using the <b>EU (Ireland)</b> region.
<IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_9.png?raw=true"><br>
  2. Open the Lambda function, starting with <b>“VoiceAlexaSkillFull-AlexaMetricSkill-1”</b> deployed with the Cloudformation.   <b>Configure your trigger</b>. Click the <b>Triggers</b> tab. Within the <b>Triggers</b> pane, click the link to <b>Add a Trigger</b>. A pop-up should appear, click in the dashed box and select Alexa Skills Kit from the list. If you don't see Alexa Skills Kit in the list, jump back to step #3 on this page.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_10.png?raw=true"><br>
  3. Once you have selected Alexa Skills Kit, click the <b>Configuration</b> Tab to go back to your code.<br>
  4. The <b>ARN value</b> should be in the top right corner. Copy this value for use in the next section of the guide.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_11.png?raw=true"><br>
</p>
  5.  Within the Lambda configuration, navigate to <b>Environment Variables</b>.  Add a greeting and exit message for your Alexa skill by adding two environment variables(case sensitive): <b>greeting_msg</b> and <b>exit_msg</b>
  <details>
<summary>Example</summary><p>
  greeting_msg <i>Welcome to the Voice Powered Analytics.  Please tell me what metrics you'd like to hear. To hear available metrics, ask Alexa tell me my metrics</i> <br>
  and
  exit_msg <i>Thank you for trying the Voice Powered Analytics.  Have a nice day!</i>
</p></details>
  6.  We'll also add an environment variable called: <b>metrics_table</b> called <i>VPA_Metrics_Table</i>.  This references the DynamoDB table that the Alexa skill will be querying for your metric 
<details>
<summary>Hint</summary><p>
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_11.png?raw=true">
  </p></details>
Optionally, you can deploy the following CloudFormation (It is neccessary to still configure : 
<table>
<thead>
<tr>
<th>Region</th>
<th>Launch Template</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>Ireland</strong> (eu-west-1)</td>
<td> <a href="https://console.aws.amazon.com/cloudformation/home?region=eu-west-1#/stacks/new?stackName=VoiceAlexaSkillFull&templateURL=https://s3.amazonaws.com/cf-templates-kljh22251-eu-west-1/skill_template.yaml"><img src="/media/images/CFN_Image_01.png" alt="Launch Alexa Skill into Ireland with CloudFormation" style="max-width:70%;"></a></td></tr></tbody></table>


</details>

### Connecting Your Voice User Interface to Your Lambda Function
On Step 1 "Setting up Your Voice User Interface" of this Step, we created a voice user interface for the intents and utterances we expect from our users. On "Step 2 Configure Alexa Backend", we created a Lambda function that contains all of our logic for the skill. On this page, we need to connect those two pieces together.<br>
<details>
<summary><strong>Full solution - Connecting VUI to Lambda (expand for details)</strong></summary><p>
  
1.  Go back to the <b><a href="https://developer.amazon.com/edw/home.html#/skills/list">Amazon Developer Portal</a></b> and select your skill from the list. You may still have a browser tab open if you started at the beginning of this tutorial.
2. Open the "Configuration" tab on the left side.<br>
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_12.png?raw=true"><br>
3. Select the <b>"AWS Lambda ARN"</b> option for your endpoint. You have the ability to host your code anywhere that you would like, but for the purposes of simplicity and frugality, we are using AWS Lambda. <a href="https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/developing-an-alexa-skill-as-a-web-service">(Read more about Hosting Your Own Custom Skill Web Service.)</a> With the AWS Free Tier, you get 1,000,000 free requests per month, up to 3.2 million seconds of compute time per month. Learn more at <a href="https://aws.amazon.com/free/">https://aws.amazon.com/free/</a>. In addition, Amazon now offers <a href="https://developer.amazon.com/alexa-skills-kit/alexa-aws-credits">AWS Promotional Credits for developers who have live Alexa skills that incur costs on AWS related to those skills</a> IMPORTANT: Make sure you select the same region that you created your Lambda in.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_13.png?raw=true"><br>
4.  Paste your <b>Lambda's ARN</b> (Amazon Resource Name) into the textbox provided. It should look similar to the screenshot above.
5.  Leave <b>"Account Linking" set to "No"</b>. For this skill, we won't be using Account Linking, but you can learn more about <a href="https://developer.amazon.com/public/solutions/alexa/alexa-skills-kit/docs/linking-an-alexa-user-with-a-user-in-your-system">Linking an Alexa User with a User in Your System.</a>
6.  Click the <b>"Next"</b> button to continue to page #4 of this guide.
</details>
</p>

### Testing Your Alexa Skill
You've now created a Voice User Interface and a Lambda function, and connected the two together. Your skill is now ready to test.
<details>
<summary><strong>Full Solution - Testing Your Alexa Skill</strong></summary><p>
1.  Go back to the <b><a href="https://developer.amazon.com/edw/home.html#/skills/list">Amazon Developer Portal</a></b> and select your skill from the list. You may still have a browser tab open if you started at the beginning of this tutorial.
2. Open the <b>"Test"</b> tab on the left side.
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_15.png?raw=true"><br>
3.  Test your skill with the <b>Service Simulator</b>. To validate that your skill is working as expected, use the Service Simulator. In the <b>Enter Utterance</b> text box, type "What’s my reinvent tweets over the last hour."
  <IMG SRC="https://github.com/awslabs/voice-powered-analytics/blob/master/media/images/Alexa_Lab_16.png?raw=true"><br>
  </p>
4.  Other testing methods to consider:
- <a href="https://echosim.io/">Echosim.io</a> - a browser-based Alexa skill testing tool that makes it easy to test your skills without carrying a physical device everywhere you go.
- <a href="https://github.com/alexa/skill-sample-nodejs-city-guide/blob/master/unit-testing.md"> Unit Testing with Alexa</a> - a modern approach to unit testing your Alexa skills with <a href="http://getpostman.com/">Postman</a> and <a href="http://aws.amazon.com/apigateway">Amazon API Gateway</a>.
5. If your sample skill is working properly, you can now customize your skill.

<summary><strong>Service Simulator Tips</strong></summary><p>
- After you click the <b>"Ask [Your Skill Name]"</b> button, you should see the <b>Lambda Request and Lambda Response boxes</b> get populated with JSON data like in the screenshot above.
- Click the <b>Listen</b> button in the bottom right corner to hear Alexa read the response.
- You can have an entire conversation with your skill with the Service Simulator. Try the following commands:
- "tell me about this place"
- [Press the listen button, and type "recommend an attraction" in the box]
- [Press the listen button, and type "give me an activity" in the box]
(Continue this process for all of the utterances. To start over, click the "Reset" button.)
- If you receive a response that reads: <i>"The remote endpoint could not be called, or the response it returned was invalid,"</i> this is an indication that something is broken. AWS Lambda offers an additional testing tool to help you troubleshoot your skill.
</p></p></details></details>


