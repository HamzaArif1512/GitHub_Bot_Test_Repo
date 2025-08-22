# GitHub ChatBot Test Repository
When working on a project, communication is key between fellow collaborators. Especially on a deadline, it is important for all team members to know who made the latest commit, where the commit was made, what the comment was, and most importantly, **what changes were made.** The demo repository tests the n8n workflow deployed on cloud software, which enables automatic push notifications. With the availability of a Gemini node on n8n, we can now have a formatted text message on what changes were made to the repository.

# Files

The initial test files in the repository consist of an html file **index.html** and a css file **style.css**.

# Testing

## Set Up
To ensure this works for multiple team members, you will have to deploy your n8n server on a cloud platform for live internet hosting.

**If you prefer not to spend, you can refer to a repository created by one of our team members. It provides a step-by-step guide on hosting an n8n server using Cloudflare:**  
 [N8N-Server by Abdullah Khan Sherwani](https://github.com/Abdullah-Khan-Sherwani/N8N-Server)

## Creation of App on Slack
The workflow uses a webhook node to retrieve all the commit information. This ranges from the author, the name of the repository, the datetime of the commit to the important URLs associated with the repository. For this workflow, we are concerned with the author, repository, commit message, and a compare URL, which we will get to later.

First, to ensure you get messages sent to you on Slack, go ahead and create an app [here](https://api.slack.com/).

Once this is done, link the app to your desired workplace and to your Slack node on your n8n workflow. 

To link the app to your desired channels in your workplace, follow the steps below:

1.  Go to your app’s settings on the Slack API dashboard.
    
2.  Under **OAuth & Permissions**, scroll down to **Scopes**.
    
    -   Add the following **Bot Token Scopes** (at minimum):
        
        -   `chat:write` → to send messages
            
        -   `channels:join` → to allow the bot to join channels
            
        -   `channels:read` → to view channels in the workspace
            
3.  Install the app to your workspace (this will generate a **Bot User OAuth Token**). Also specify the channel(s) you want your bot to operate in.
    
4.  Copy the OAuth token and paste it into the **Slack node credentials** in your n8n workflow.
    
5.  In Slack, invite your bot to the desired channel using:
    **/invite @your-app-name**


## Connect Compare File to Gemini

Displaying the author, message, and repository is easy if you connect the webhook to the slack node directly. We can improve the experience by adding Gemini to the workflow to tell us where the changes were made. To do this, we require the **.diff** file from the compare URL. To do this we run a HTTP request node on the compare URL from the webhool and add two parameters:

 1. One for accessing the compare file by creating a GitHub classic
    token (especially if your repo is private).
 2. One for accessing the .diff file, which is the raw file outlining the
    changes between the  previous and current commit.

This part is tricky and requires several push test attempts to debug why the connection is not being made. While creating the token, make sure you have given repo access. If it still does not work, I suggest trying other access features.

After this, connect your HTTP Request node (containing the compare URL) to a Gemini node. For the gemini node, you are required to make a free Google API key [here](https://aistudio.google.com/app/apikey). Use this to connect to the Gemini node.  Now your Gemini node is ready to receive prompts in your n8n workflow.

If your HTTP Request is successful, you should get the raw contents of the .diff file. Pass this as a parameter to Gemini with your preferred model and prompt. Pass the output to the Slack node, and you are done. Make sure to review Gemini's message to ensure it is giving the desired response. Otherwise, play around with the model and change the prompt message for a better answer.

# Connect Webhook to GitHub
In the GitHub webhook settings of your repository, create a new webhook and enter the payload URL, which goes something like this:

https://<your-domain.com>/webhook/github-events

Make sure to:

1.  Set the **Content type** to `application/json`.
    
2.  Choose **Just the push event** (or select individual events you want to listen for).
    
3.  Enable the webhook by clicking **Add webhook**.
    

Once done, GitHub will send a test payload to your endpoint, and you should see it appear in your n8n workflow if everything is set up correctly.

# Workflow


The final n8n workflow consists of **four nodes** connected in sequence:


1. **Webhook Node**  
   - Listens for incoming events from GitHub (via the repository webhook).  

2. **HTTP Node (Compare URL)**  
   - Fetches and compares the commit or PR details from the GitHub payload.  

3. **Gemini Node**  
   - Processes or summarizes the commit message/content using Gemini.  

4. **Slack Node (Send a Message)**  
   - Sends the processed output directly into the configured Slack channel.  

---

 With this setup, every new commit or PR triggers the webhook, gets processed through Gemini, and a neatly formatted update is posted to Slack automatically.


