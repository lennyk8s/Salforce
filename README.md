# Salforce

This repo contains all setup instructions and changes made to my Salesforce environment managed by Salto. 


** SETUP **

** 1. Set Up Your Environment **

Ensure you have Salto CLI installed and configured for your Salesforce environment.

Connect Salto to your Salesforce instance

  salto login salesforce

Set up your Git repository and clone it locally

  git clone git@github.com:lennyk8s/Salforce.git
  cd Salforce
  
NOTE: In certain enviornments you may need to ensure you have your ssh credentials created appropriately both locally and in Github. Please see:
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
https://docs.github.com/en/authentication/connecting-to-github-with-ssh/adding-a-new-ssh-key-to-your-github-account

** 2. Fetch and Extract Salesforce Configuration **

Use Salto to fetch metadata from Salesforce  
  salto fetch -e env1 

Note: env1 represents my Salto Salesforce Workspace

** 3. Create a Feature Branch **

Create a new branch for your changes  
 
  git checkout -b feature/<your-feature-name>
   
NOTE: This keeps the main branch clean and allows for better tracking.

** 4. Modify Metadata and Deploy Locally **

Make your metadata changes using Salto or Salesforce UI.

NOTE: If changes were made outside of Salto, fetch them again   
  
 salto fetch -e env1
   
Review the modifications  
 
 git diff
   
** 5. Commit and Push Changes **

Stage the changes

 git add .
   
Write a meaningful commit message
 
 git commit -m "Added/Modified [feature] using Salto"
   
Push to the remote repository
 
 git push origin feature/<your-feature-name>
   
** 6. Open a Pull Request (PR) **

Open a PR to merge your changes into the main branch to ensure the team reviews the changes and approvals are received.

** 7. Merge and Deploy to Production **

Once approved, merge the PR into main

 git checkout main
 git pull origin main
 git merge feature/<your-feature-name>
 git push origin main
   

If there is a Production environemnt that the changes need to be promoted to, then  continue with the following directions

** 8. Deploy to Salesforce production **  

 salto deploy -e <prod-salesforce-env>
   
Perform post-deployment validation.

** 9. Cleanup **

Delete the feature branch after merging  

 git branch -d feature/<your-feature-name>
 git push origin --delete feature/<your-feature-name>


** Changes **

1. Created Hot and Cold Queues for Leads - 02/20/2025

Queues were created in Salesforce directly.

2. Created Lead Assignment Rules for Hot Queue Leads and Cold Queue Leads. Leads are redirected by a custom field labeled "score" where a lead with a score of >=75 is
considered a Hot Lead and a score of <=74 is considered a Cold Lead.

The following files were edited with the following information to make the changes:

Custom "score" field

a. Edit the LeadCustomFields.nacl object file 
b. Added the following code:

salesforce.Text Score__c {
    apiName = "Lead.Score__c"
    externalId = false
    label = "Score"
    length = 80
    unique = false
    createable = true
    updateable = true
    caseSensitive = false
    queryable = true
  }

Lead Assignment Rules for Hot Queue Leads and Cold Queue Leads

a. Edit the Lead.nacl object file under the AssignmentRules directory
b. Added the following code:

salesforce.AssignmentRules Lead {
  assignmentRule = [
    {
      fullName = "Lead Score Based Assignment Rule"
      active = true
      ruleEntry = [
        {
          assignedTo = salesforce.Queue.instance.Hot_leads
          assignedToType = "Queue"
          criteriaItems = [
            {
              field = salesforce.Lead.field.Score__c
              operation = "greaterOrEqual"
              value = "75"
            },
          ]
        },
        {
          assignedTo = salesforce.Queue.instance.Cold_Leads
          assignedToType = "Queue"
          criteriaItems = [
            {
              field = salesforce.Lead.field.Score__c
              operation = "lessOrEqual"
              value = "74"
            },
          ]
        },
      ]
    },
    
