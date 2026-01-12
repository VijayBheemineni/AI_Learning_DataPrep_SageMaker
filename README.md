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