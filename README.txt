================================================================================
REGULATORY & COMPLIANCE AUTOMATION SYSTEM - README
Challenge CH-04: Automated Compliance Analysis Using RAG
================================================================================


PROJECT OVERVIEW:
This solution implements an automated financial compliance analysis system
using Retrieval-Augmented Generation (RAG) architecture.

The n8n workflow automates compliance assessment for business cases across multiple jurisdictions (UAE, KSA, EU, EG). The system uses RAG (Retrieval-Augmented Generation) to search company policies and regulatory databases, then provides compliance decisions through an AI agent.

EXPECTED RUN TIME: 10-30 seconds per case (depending on complexity and API response times)

========================================================================================================
PREREQUISITES
========================================================================================================
1. Computer with at least 8GB RAM (16GB recommended for Ollama)
2. Stable internet connection
3. Basic command line knowledge
4. Windows 10/11, macOS 10.15+, or Linux (Ubuntu 20.04+)

========================================================================================================
PART 1: INSTALL N8N (Workflow Automation Platform)
========================================================================================================

METHOD A - Using NPM (Recommended for most users):
1. Install Node.js 18.x or 20.x from https://nodejs.org/
2. Open Terminal/Command Prompt and verify installation:
   node --version
   npm --version
3. Install n8n globally:
   npm install -g n8n
4. Start n8n:
   n8n
5. Open browser and navigate to: http://localhost:5678
6. Create your n8n account (email + password)

METHOD B - Using Docker (For advanced users):
1. Install Docker Desktop from https://www.docker.com/products/docker-desktop
2. Run in terminal:
   docker run -it --rm --name n8n -p 5678:5678 -v ~/.n8n:/home/node/.n8n n8nio/n8n
3. Open browser: http://localhost:5678

METHOD C - Desktop App (Easiest for beginners):
1. Download n8n Desktop from https://n8n.io/desktop
2. Install and launch the application
3. The interface will open automatically

========================================================================================================
PART 2: INSTALL OLLAMA (Local AI Models)
========================================================================================================

STEP 1 - Install Ollama:
Windows:
1. Download from https://ollama.ai/download/windows
2. Run the installer (OllamaSetup.exe)
3. Follow installation wizard

macOS:
1. Download from https://ollama.ai/download/mac
2. Open the .dmg file and drag Ollama to Applications

Linux:
1. Run in terminal:
   curl -fsSL https://ollama.ai/install.sh | sh

STEP 2 - Verify Installation:
Open terminal and run:
ollama --version

STEP 3 - Download Required Models:
Run these commands one by one:
ollama pull bge-m3:latest
ollama pull qwen2.5:3b

Expected download time: 5-15 minutes depending on internet speed
Expected disk space: ~4GB total

STEP 4 - Verify Models:
ollama list

You should see both models listed.

STEP 5 - Configure Ollama for n8n:
By default, Ollama runs on http://localhost:11434
Keep Ollama running in the background (it starts automatically after installation)

========================================================================================================
PART 3: SETUP PINECONE (Vector Database)
========================================================================================================

STEP 1 - Create Pinecone Account:
1. Go to https://www.pinecone.io/
2. Click "Start Free" 
3. Sign up with email or Google account
4. Verify your email address

STEP 2 - Create API Key:
1. Login to Pinecone dashboard
2. Go to "API Keys" section in left sidebar
3. Click "Create API Key"
4. Copy the key and save it securely (you'll need it later)

STEP 3 - Create Index:
1. Click "Indexes" in left sidebar
2. Click "Create Index"
3. Configure as follows:
   - Index Name: rules-eva
   - Dimensions: 1024
   - Metric: cosine
   - Environment: Starter (free tier)
4. Click "Create Index"
5. Wait 1-2 minutes for index to be ready

IMPORTANT: The index name MUST be "rules-eva" to match the workflow configuration.

========================================================================================================
PART 4: SETUP GROQ (AI Chat Model)
========================================================================================================

STEP 1 - Create Groq Account:
1. Go to https://console.groq.com/
2. Click "Sign Up" and create account
3. Verify your email

STEP 2 - Get API Key:
1. Login to Groq Console
2. Navigate to "API Keys" section
3. Click "Create API Key"
4. Name it "n8n-compliance" (or any name)
5. Copy the API key and save it securely

Note: Groq offers free tier with good rate limits for testing.

========================================================================================================
PART 5: IMPORT WORKFLOW INTO N8N
========================================================================================================

STEP 1 - Open n8n:
Navigate to http://localhost:5678 in your browser

STEP 2 - Import Workflow:
1. Click "Workflows" in left sidebar
2. Click "+ Add Workflow" button (top right)
3. Click the three dots menu (⋮) > "Import from File"
4. Select the "main.json" file from your project folder
5. Click "Import"

STEP 3 - Configure Credentials:

A) Pinecone Credentials:
1. Click on any "Pinecone" node in the workflow
2. Click "Select Credential" dropdown
3. Click "+ Create New Credential"
4. Enter:
   - Credential Name: PineconeApi account
   - API Key: [Paste your Pinecone API key from Part 3]
   - Environment: us-east-1 (or your region)
5. Click "Save"

B) Groq Credentials:
1. Click on "Groq Chat Model" node
2. Click "Select Credential" dropdown  
3. Click "+ Create New Credential"
4. Enter:
   - Credential Name: Groq account
   - API Key: [Paste your Groq API key from Part 4]
5. Click "Save"

C) Ollama Credentials:
1. Click on any "Ollama" node
2. Click "Select Credential" dropdown
3. Click "+ Create New Credential"
4. Enter:
   - Credential Name: Ollama account
   - Base URL: http://localhost:11434
5. Click "Save"
6. Apply this credential to all Ollama nodes in the workflow

========================================================================================================
PART 6: PREPARE DATA FILES
========================================================================================================

STEP 1 - Create Data Folder:
Create a folder structure on your computer:
/compliance-data/
  ├── company_policies.jsonl
  └── regulations.jsonl

STEP 2 - Upload to Google Drive (for VD workflow):
1. Upload both JSONL files to your Google Drive
2. Note the file paths for later

OR

STEP 2 - Use Local Files (alternative):
1. In n8n workflow, replace "Search files and folders" node with "Read Binary Files" node
2. Point to your local file paths

========================================================================================================
PART 7: INDEX YOUR DATA (First-time Setup)
========================================================================================================

STEP 1 - Activate VD Workflow:
1. In n8n, locate the "VD" section (left side of workflow)
2. This workflow indexes your policies and regulations into Pinecone

STEP 2 - Configure File Source:
1. Update "Search files and folders" node with your Google Drive paths
2. OR configure local file paths if using local files

STEP 3 - Run Indexing:
1. Click "Execute Workflow" button
2. Watch the execution flow (green lines indicate success)
3. Expected time: 2-5 minutes for ~150 rules

STEP 4 - Verify Indexing:
1. Go to Pinecone dashboard
2. Click on "rules-eva" index
3. Check "Vector Count" - should show ~150 vectors
4. Check namespaces: "company_policies.json" and "regulations.json" should exist

========================================================================================================
PART 8: RUN THE COMPLIANCE AGENT
========================================================================================================

STEP 1 - Activate Chat Interface:
1. In n8n workflow, click on "When chat message received" node
2. Click "Settings" tab
3. Copy the "Production URL" 
4. The URL will look like: http://localhost:5678/webhook/[unique-id]

STEP 2 - Open Chat Interface:
1. Paste the Production URL in a new browser tab
2. You'll see a chat interface with welcome message

STEP 3 - Test with Sample Case:
Copy and paste this test case:

case_id: CASE001
case_type: complaint_response
jurisdiction: UAE
risk_level: high
description: Customer complaint about product quality

STEP 4 - Review Results:
The agent will:
1. Search company policies (2-5 seconds)
2. Search regulations (2-5 seconds)
3. Analyze compliance (5-10 seconds)
4. Return decision in format: "[Status] - [Requirement]"

Expected total time: 10-30 seconds

STEP 5 - Test Different Case Types:
Try these case types:
- complaint_response
- ad_copy
- label_change
- ingredient_change
- data_request

Each with different jurisdictions: UAE, KSA, EU, EG

========================================================================================================
TROUBLESHOOTING
========================================================================================================

ISSUE: "Ollama connection failed"
SOLUTION: 
- Ensure Ollama is running (check system tray/menu bar)
- Verify URL is http://localhost:11434
- Restart Ollama service

ISSUE: "Pinecone namespace not found"
SOLUTION:
- Re-run VD workflow to index data
- Check index name is exactly "rules-eva"
- Verify namespace names match file names

ISSUE: "Groq API rate limit"
SOLUTION:
- Wait 60 seconds and retry
- Check your Groq dashboard for quota
- Consider upgrading Groq plan

ISSUE: "Workflow execution timeout"
SOLUTION:
- Increase timeout in n8n settings (Settings > Executions > Timeout)
- Reduce topK value in vector search nodes (default: 5)

ISSUE: "Empty response from agent"
SOLUTION:
- Check system prompt is properly configured
- Verify both tools are connected to AI Agent
- Review execution logs for errors

========================================================================================================
PERFORMANCE OPTIMIZATION
========================================================================================================

1. For faster responses:
   - Reduce topK from 5 to 3 in vector search tools
   - Use smaller Ollama model (qwen2.5:1.5b instead of 3b)

2. For better accuracy:
   - Increase topK to 10
   - Use larger Groq model (llama-3.1-70b instead of gpt-oss-120b)

3. For batch processing:
   - Use "Loop Over Items" node to process multiple cases
   - Import cases.csv and connect to workflow

========================================================================================================
MAINTENANCE
========================================================================================================

1. Update regulations monthly:
   - Edit regulations.jsonl file
   - Re-run VD workflow to re-index
   - Pinecone will update automatically

2. Monitor usage:
   - Groq: Check dashboard for API usage
   - Pinecone: Monitor vector count and queries
   - n8n: Review execution history for errors

3. Backup:
   - Export workflow regularly (Workflow > Export)
   - Keep copies of JSONL files
   - Note: Pinecone data is cloud-backed

========================================================================================================
SUPPORT AND RESOURCES
========================================================================================================

n8n Documentation: https://docs.n8n.io/
Ollama Documentation: https://github.com/ollama/ollama
Pinecone Documentation: https://docs.pinecone.io/
Groq Documentation: https://console.groq.com/docs/

For project-specific issues, check execution logs in n8n workflow view.

========================================================================================================
END OF SETUP GUIDE
========================================================================================================