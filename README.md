# AWS SageMaker Canvas Data Preparation Guide

I've been working with SageMaker Canvas for data prep lately, and honestly, it's pretty powerful once you get it set up right. This guide walks through everything I've learned about getting your data ready for ML models using Canvas's visual interface - no coding required.

## ⚠️ **Heads up - Canvas can get expensive fast!** ⚠️

I learned this the hard way - Canvas costs about **$1.90/hour** (as of Jan 2026, US regions). That might not sound like much, but it adds up quick if you forget to log out.

**Here's what you're paying for:**
- **Workspace time**: ~$1.90/hr just for having Canvas open
- **Model training**: Extra compute costs when building models  
- **Predictions**: Costs for running inference
- **AI services**: Additional charges if you use ready-made models

**The big gotcha**: Canvas keeps running (and billing) even if you just close your browser! I've seen people rack up hundreds in charges this way.

### How to actually stop the billing:

**The right way (do this!):**
1. Click the logout button in Canvas (left sidebar)
2. Wait for the "relaunch in different tab" message
3. Now you can close your browser safely

**Emergency stop (if you forgot to log out):**
1. Go to SageMaker Console → Domains
2. Find your domain → User profiles  
3. Click your user → Apps tab
4. Hit "Delete" next to the Canvas app
5. Confirm - billing stops immediately

**For teams/admins:**
You can set up Lambda functions to auto-shutdown idle Canvas apps. There are CloudFormation templates available that make this pretty straightforward.

**Pro tips:**
- Set up billing alerts - seriously, do this first
- Canvas charges show up as `REGION-Canvas:Session-Hrs` in your bill
- Check current pricing at [aws.amazon.com/sagemaker/canvas/pricing](https://aws.amazon.com/sagemaker/canvas/pricing/) since it changes

---

## Task 1: Setting up your SageMaker Domain

### Why you need a SageMaker Domain first

Before you can use Canvas, you need to create what AWS calls a "SageMaker Domain." Think of it like setting up a workspace - it's where all your ML stuff lives and gets managed.

Here's what the domain actually does:
- **Manages your resources**: Controls who can access what compute, storage, etc.
- **Keeps things secure**: Creates boundaries between different teams/projects  
- **Hosts your apps**: Canvas (and other SageMaker tools) need this to run
- **Handles users**: Lets you create different user profiles with different permissions
- **Stores your work**: Decides where your models, datasets, and other files go

Bottom line: No domain = no Canvas. It's just how SageMaker works.

### User profiles - why you need them too

Once you have a domain, you need user profiles. Each person gets their own profile, which gives them:
- Their own workspace (so you don't mess with each other's stuff)
- Specific permissions (maybe some people can deploy models, others can't)
- Personal settings and configurations
- Individual cost tracking (helpful for billing)

### What you need before starting

Make sure you have:
- An AWS account (obviously)
- Permissions to create SageMaker domains (check with your admin if unsure)
- Optionally: your own VPC if you're doing this for production
- An S3 bucket where Canvas can store your data

### Setting up your SageMaker Domain

Here's how to set up a custom domain with full control over permissions and settings:

1. **Start from the Domains page**
   - Go to [console.aws.amazon.com/sagemaker](https://console.aws.amazon.com/sagemaker)
   - Click "Domains" in the left sidebar
   - Click "Create domain"

2. **Choose custom setup**
   - Select "Custom setup" for advanced options
   - You'll get way more configuration choices

3. **Basic domain config**
   - Give it a name that makes sense
   - **Authentication method**: Select "IAM" (this is what we're using)
   - Pick your VPC and subnets (or use defaults)
   - Set up encryption if you need it

4. **Pick your Canvas permissions**
   - You can select up to 8 "ML activities" total
   - **Must have**: "Run Studio Applications" and "Canvas Core Access"
   - **For data prep**: "Canvas Data Preparation" (that's why we're here!)
   - **Optional stuff**: AI Services, MLOps, Kendra access

5. **Storage settings**
   - **System managed**: AWS creates an S3 bucket for you
   - **Customer managed**: Use your own S3 bucket

### Creating your user profile

After the domain is ready:

1. **Go to your domain**
   - SageMaker Console → Domains → click your domain name

2. **Add a user**
   - Click "Add user"
   - Pick a username
   - **Execution role**: Select the default "AmazonSageMaker-ExecutionRole-*" that already exists in your account (this has all the permissions you need)

3. **Set permissions**
   - Usually just inherit from the domain settings
   - Customize if you need different access levels

### What Canvas actually needs to work

**The essentials:**
- Core Canvas access (obviously)
- Data preparation permissions (for the data wrangling we want to do)
- S3 access (where your data lives)
- Basic networking setup

**Nice to have:**
- AI Services (for ready-made models)
- MLOps features (if you want to deploy models later)
- Database connectors (Redshift, RDS, etc.)

### Testing everything works

Once you're set up:

1. **Try launching Canvas**
   - Go to SageMaker Console → Canvas
   - Click "Open Canvas" next to your user
   - Should launch in a new tab

2. **Quick permission check**
   - See if you can access the data import features
   - Try creating a new dataset
   - Make sure you can see the transformation tools

3. **Set up cost monitoring** (seriously, do this)
   - Turn on AWS Cost Explorer
   - Create billing alerts
   - Add cost tags if you're sharing the account

### When things go wrong

**Domain won't create:**
- Check your IAM permissions
- Make sure your VPC/subnet setup is valid
- Verify Canvas is available in your region

**Canvas won't launch:**
- Double-check the user profile has Canvas permissions
- Make sure the execution role has the right policies
- Confirm domain status shows "InService"

**Surprise billing:**
- If Canvas is still running, force-stop it via the console
- Set up auto-shutdown (Lambda functions work great)
- Use Cost Explorer to see where charges are coming from

**Canvas keeps billing after closing browser:**
This is the most common mistake! Canvas doesn't stop when you close the browser.
- **Fix**: Go to Console → Domains → Your Domain → User Profiles → Apps → Delete the Canvas app
- **Prevent**: Always use the logout button in Canvas first

### What's next

Once your domain and user are working:
- Task 2: Getting your data into Canvas
- Task 3: Checking data quality  
- Task 4: Transforming and cleaning data
- Task 5: Exporting your final dataset

---

**Remember: Canvas costs $1.90/hour even when you're not actively using it. Always log out properly!**

## Task 2: Getting your data into Canvas with Data Wrangler

### What is Data Wrangler?

Data Wrangler is basically Canvas's built-in data prep tool - think of it as your visual data cleaning assistant. Instead of writing code to clean and transform your data, you get a drag-and-drop interface where you can:

- Import data from various sources (S3, databases, etc.)
- See what your data looks like with automatic profiling
- Clean up messy data with point-and-click transformations
- Create a visual "flow" that shows exactly what you did to your data
- Export the cleaned data for model training

The cool thing is that Data Wrangler creates a visual pipeline of all your data prep steps, so you can see exactly what happened to your data and easily reproduce it later.

### Importing CSV data from S3

For this task, I'm assuming you already have a CSV file sitting in an S3 bucket. If you don't, just upload one through the S3 console first.

Here's how to get your data from S3 into Canvas for cleaning:

1. **Launch Canvas and find Data Wrangler**
   - Open Canvas from your SageMaker console (like we did in Task 1)
   - On the Canvas home page, look for "Import and prepare" 
   - Click on it - this launches Data Wrangler

2. **Start a new data flow**
   - You'll see the Data Wrangler interface open up
   - It might take a minute or two to load the first time
   - You should see a "Create data flow" option or it might start automatically
   - **Name your data flow**: Change the default name to "VijayDataPrepPoC" so it's easy to find later

3. **Connect to your S3 data**
   - Look for "Import data" or a similar button
   - Choose "Amazon S3" as your data source
   - You'll see a file browser showing your S3 buckets

4. **Navigate to your CSV file**
   - Browse through your S3 buckets to find your CSV file
   - Click on the bucket name, then navigate through folders if needed
   - When you find your CSV file, click on it

5. **Preview and configure the import**
   - Data Wrangler will show you a preview of your CSV
   - Check if the column headers look right
   - Make sure the data types look reasonable (it auto-detects them)
   - You can adjust settings like:
     - Whether the first row contains headers
     - What delimiter is used (comma, semicolon, etc.)
     - How to handle missing values

6. **Import the data**
   - Once the preview looks good, click "Import" or "Create dataset"
   - Data Wrangler will pull in your data and create the first step in your data flow
   - You'll see your dataset appear as a node in the visual flow

### What happens after import

Once your data is imported, Data Wrangler automatically:
- **Analyzes your data**: Shows you statistics, missing values, data distributions
- **Infers data types**: Tries to figure out if columns are numbers, text, dates, etc.
- **Creates a data flow**: Your import becomes the first step in a visual pipeline

You'll see something like a flowchart with your imported dataset as the starting point. From here, you can add transformation steps, analyze data quality, and prepare your data for ML.
## Task 3: Checking your data quality

Now that we've got our data imported into Data Wrangler, it's time to actually look at what we're working with. This is probably the most important step - you can't build good models with bad data, so we need to understand what issues we're dealing with.

### 3a: Quick data validation using the Data tab

First, let's do a basic sanity check of our data:

1. **Look at the Data tab**
   - In your Data Wrangler flow, you should see your imported dataset as a node
   - Click on that node to select it
   - Look for a "Data" tab (usually at the bottom or side of the interface)
   - Click on it to see your actual data

2. **Scan through your data**
   - This shows you the raw data in a table format
   - Scroll through a few rows to get a feel for what you're working with
   - Look for obvious issues like:
     - Columns that are completely empty
     - Weird characters or formatting issues
     - Data that doesn't look right (like negative ages or impossible dates)
     - Missing values showing up as blanks, "NULL", "N/A", etc.

3. **Check the column headers**
   - Make sure all your expected columns are there
   - Verify the column names make sense
   - Look at the data types (usually shown at the top of each column)

This gives you a quick gut check - does the data look roughly like what you expected?

### 3b: Deep dive with Data Quality and Insights Report

Now for the real analysis. Data Wrangler has a built-in tool that automatically analyzes your data and spots issues:

1. **Start the analysis**
   - Right-click on your dataset node (or look for a "+" or ellipsis icon next to it)
   - Choose "Get data insights" or "Add analysis"
   - You'll see analysis options appear

2. **Create the Data Quality and Insights Report**
   - For "Analysis type", select **"Data Quality and Insights Report"**
   - Give it a name: **"VijayPoCDataQualityAndInsights"**
   - For "Target column", select **"Income"**
   
   **What's a target column in AI?** Think of it as the "answer" you want your AI model to predict. In machine learning, we use historical data to teach the computer to make predictions. The target column (also called the "label" or "dependent variable") is what we're trying to predict based on all the other columns (called "features").
   
   For example:
   - If you want to predict house prices, "Price" would be your target column
   - If you want to predict if an email is spam, "IsSpam" would be your target
   - In our case, we're using "Income" as the target - so we're building a model that can predict someone's income based on their other characteristics
   
   The target column helps Data Wrangler understand which column is most important and analyze how well the other columns can help predict it.

   - For "Problem type", choose either:
     - **Regression** if you're trying to predict a number (like price, temperature, etc.)
     - **Classification** if you're trying to predict categories (like yes/no, spam/not spam, etc.)
   
   **Quick rule**: If your target column data type is "String", choose **Classification**. If it's "Number", choose **Regression**.
   
   **For our Income example**: Since our Income column contains categories ("<=50K" or ">50K") rather than actual dollar amounts, we should select **"Classification"**. Even though we're dealing with income, the data is formatted as categories, not continuous numbers.

3. **Choose your data size**
   - Select **"Sampled dataset"** (this is the recommended default)
   - **Sampled dataset**: Uses up to 200,000 rows (faster, good for initial exploration)
   - **Full dataset**: Analyzes everything (slower, but more complete - costs extra compute time)
   - For most cases, "Sampled dataset" is perfect to start with

5. **Run the analysis**
   - Click "Create" and wait for it to finish
   - This usually takes a few minutes depending on your data size
   
   **⚠️ ERROR**: Sometimes you might get an error like **"Something went wrongOperatorCustomerError"**. This often happens when you select the wrong problem type - for example, choosing "Regression" when your target column contains categories instead of numbers. If you get this error, try switching between "Regression" and "Classification" to see which one works with your data.
   
   **Problem Type Tip**: If you get errors with "Regression" but "Classification" works, check your target column's data type. In our case, Income contains categories ("<=50K", ">50K") rather than actual numbers, so Classification is the right choice even though we're dealing with income data.

### What the report tells you

Once it's done, you'll get a comprehensive report that shows:

**Summary section:**
- How many missing values you have
- Number of outliers (weird extreme values)
- Data types for each column
- Overall data quality score

**Target analysis (if you picked a target):**
- How balanced your target variable is
- Distribution of values you're trying to predict
- Potential issues with your target

**Feature insights:**
- Which columns are most useful for prediction
- Columns that might be causing problems
- Recommendations for data cleaning

**Quick model results:**
- A rough estimate of how well a model might perform on this data
- Helps you understand if your data is even suitable for ML