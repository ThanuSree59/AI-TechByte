# TechByte Outlook Workflow

An intelligent n8n automation workflow that curates trending YouTube AI/tech content and sends AI-generated summaries to your email for approval. The workflow fetches videos from top tech influencers, analyzes engagement metrics, generates concise summaries using Groq AI, and manages email approvals with revision capabilities.

## 📋 Features

- **Automated Content Curation**: Fetches latest videos from 8 top tech/AI influencers
- **Engagement Analytics**: Calculates engagement scores based on views, likes, and comments
- **Transcript Extraction**: Automatically retrieves video transcripts using RapidAPI
- **AI-Powered Summaries**: Generates human-friendly summaries using LLM
- **Email Approval Workflow**: Sends summaries to Outlook for user approval/rejection
- **Revision Support**: Allows you to request revisions without changing the video
- **Duplicate Prevention**: Uses Pinecone vector DB to prevent re-sending previous content
- **Scheduled Execution**: Runs automatically at 11 AM on weekdays (Mon-Fri)

## 📦 Prerequisites

Before running this workflow, ensure you have:

### 1. **n8n Installation**
   - n8n self-hosted or cloud instance running
   - Access to n8n editor and workflows

### 2. **API Keys & Credentials**

   - **YouTube API**: 
     - Create a project in Google Cloud Console
     - Enable YouTube Data API v3
     - Create OAuth 2.0 credentials
     - Save credentials in n8n

   - **RapidAPI Key** (YouTube Transcript):
     - Sign up at https://rapidapi.com
     - Subscribe to "YouTube Transcriptor" API
     - Obtain API key and host

   - **Groq AI (LangChain/LLM)**:
     - Groq key 
     - Configure in n8n credentials

   - **Pinecone Vector Database**:
     - Create Pinecone index at https://www.pinecone.io
     - Get API key and index URL
     - Note: Index structure should support `sent-videos` namespace

   - **Outlook/Email Integration**:
     - Office 365 OAuth credentials
     - Email address for approvals

### 3. **Hardware/Environment**
   - Stable internet connection
   - n8n server with adequate memory
   - Cron scheduler enabled (for scheduled triggers)

## 🚀 Installation Steps

### Step 1: Import the Workflow

1. Open n8n editor
2. Go to **Workflows** → **Create New** or **Import**
3. Copy the contents of `TechByte-outlook.json`
4. Click **Import** and select the file
5. The workflow will appear in your list

### Step 2: Configure API Credentials

1. **YouTube API**:
   - In n8n, go to **Credentials**
   - Create new credential: "YouTube OAuth2 API"
   - Authenticate with your Google account

2. **LLM**:
   - Create new credential: "GROQ" (depending on your LLM)
   - Use your API key 
   - Store securely

3. **HTTP Requests (RapidAPI)**:
   - Update RapidAPI key in the "Fetch Transcript" node:
     ```
     x-rapidapi-key: YOUR_RAPIDAPI_KEY
     x-rapidapi-host: youtube-transcriptor.p.rapidapi.com
     ```

4. **Pinecone Database**:
   - Update in the HTTP request nodes accessing Pinecone:
     ```
     URL: https://your-index-name.svc.region-xxxxxx.pinecone.io/vectors/fetch
     Api-Key: YOUR_PINECONE_API_KEY
     ```

5. **Outlook/Email**:
   - Create new credential: "Microsoft Outlook OAuth2"
   - Authenticate with your Outlook account
   - Grant permissions for sending emails and forms

### Step 3: Update Workflow Configuration

1. **Email Settings**:
   - Update the email address in the workflow to your Outlook email
   - Configure approval form URL/settings

2. **Influencer List** :
   - Edit the "Select Top 5 Influencers" node to add/remove YouTube channels
   - Current influencers: Cole Medin, AI Uncovered, Matt Wolfe, ByteByteGo, nicksaraev, Fireship, airevolutionx, TheAiGrid

3. **Schedule** (Optional):
   - Default: 11 AM on weekdays (0 0 11 * * 1-5)
   - Edit "Schedule Trigger" node to change timing
   - Cron format: `<seconds> <minutes> <hours> <day> <month> <day-of-week>`

### Step 4: Test the Workflow

1. Click **Test Workflow** or manually trigger "Schedule Trigger1"
2. Check execution logs for errors
3. Verify:
   - Videos are fetched from YouTube
   - Transcripts are retrieved
   - AI summary is generated
   - Email is sent with approval form

## 📧 Running the Workflow

### Automatic (Scheduled)
- Workflow runs automatically at 11 AM every weekday
- No manual intervention needed
- Check n8n execution logs for status

### Manual Trigger
1. Open the workflow in n8n editor
2. Click **Execute Workflow** button
3. Wait for completion
4. Check execution logs for any errors

### Monitoring
- Open **Executions** tab to view run history
- Check success/failure status
- Review logs for debugging

## 📨 Approval Process

When the workflow runs successfully:

1. **Email Received**: You'll receive an email with:
   - Video summary
   - Engagement metrics
   - Video link and channel info
   - Approval/Revision options

2. **Respond with**:
   - ✅ **Approve**: Summary is finalized and recorded
   - 📝 **Reject - Revise Content**: Provide feedback for revision (same video)
   - ⏭️ **Reject - Try Next Video**: Skip this video, try next one

3. **Revision Loop**: 
   - Workflow waits 2 hours for response
   - If you revise, AI regenerates based on your feedback
   - You can approve or request another revision
   - Up to 3 revision cycles before stopping

## 🔧 Troubleshooting

### Common Issues

**❌ YouTube API Error**
- Solution: Check OAuth credentials are valid and YouTube Data API is enabled

**❌ Transcript Fetch Fails**
- Solution: Verify RapidAPI key is correct and quota not exceeded

**❌ AI Summary Not Generated**
- Solution: Check LLM API key and ensure model is available

**❌ Pinecone Connection Error**
- Solution: Verify Pinecone API key, index URL, and region configuration

**❌ Email Not Received**
- Solution: Check Outlook credentials, spam folder, and email permissions in Azure

**❌ Workflow Timeout**
- Solution: Increase timeout settings in HTTP nodes; check API response times

## 📝 Workflow Structure

```
Schedule Trigger (11 AM weekdays)
    ↓
Select Top 5 Influencers (Random selection)
    ↓
Fetch Videos via YouTube API (Last 7 days)
    ↓
Extract Video Metadata
    ↓
Fetch Video Statistics (Views, Likes, Comments)
    ↓
Calculate Engagement Score & Select Top 10
    ↓
Check Pinecone (Previously Sent Videos)
    ↓
Filter Out Sent Videos
    ↓
Select First Unsent Video
    ↓
Fetch Transcript (RapidAPI)
    ↓
Process Transcript
    ↓
Generate AI Summary (Claude)
    ↓
Parse LLM Response
    ↓
Send Email with Approval Form
    ↓
Wait for User Response (2 hours timeout)
         ├─→ Approved → Record in Pinecone ✅
         ├─→ Revise → Regenerate Summary 📝
         └─→ Try Next → Loop to next video ⏭️
```

## 📊 Data Flow

```
YouTube API → Video Metadata (IDs, stats)
                    ↓
            RapidAPI YouTube Transcriptor
                    ↓
            Transcript + Metadata → Claude AI
                    ↓
            Generated Summary + Metadata
                    ↓
            Outlook Email → User Approval
                    ↓
            Pinecone (Deduplication DB)
```

## 🎯 Performance Optimization

- **Batch Processing**: Videos are processed one at a time for reliability
- **Engagement Filtering**: Keeps top 10 by engagement score
- **Dedupe Prevention**: Pinecone index prevents sending duplicates
- **Error Handling**: Failed nodes trigger retry logic

## 📚 API Integrations

| Service | Purpose | Docs |
|---------|---------|------|
| **YouTube API** | Fetch videos, statistics | https://developers.google.com/youtube/v3 |
| **RapidAPI (Transcriptor)** | Extract video transcripts | https://rapidapi.com/pyshark/api/youtube-transcriptor |
| **Claude/OpenAI LLM** | Generate summaries | https://www.anthropic.com or https://openai.com |
| **Pinecone** | Vector database for dedup | https://www.pinecone.io/docs |
| **Microsoft Outlook** | Send approval emails | https://learn.microsoft.com/en-us/graph/api/overview |

