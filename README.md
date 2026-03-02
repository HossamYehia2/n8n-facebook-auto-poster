# 📘 n8n Facebook Page Auto-Poster

Automatically post content from your dataset to your Facebook Page using **n8n** — an open-source workflow automation tool.

---

## 📋 Table of Contents

1. [What is n8n?](#what-is-n8n)
2. [Prerequisites](#prerequisites)
3. [Quick Start](#quick-start)
4. [Facebook Setup (Required)](#facebook-setup-required)
5. [How the Workflow Works](#how-the-workflow-works)
6. [Managing Your Posts Dataset](#managing-your-posts-dataset)
7. [Scheduling Options](#scheduling-options)
8. [Importing the Workflow](#importing-the-workflow)
9. [Troubleshooting](#troubleshooting)

---

## 🤔 What is n8n?

**n8n** (pronounced "nodemation") is a free, open-source workflow automation tool. It works like a visual programming interface:

- You create **workflows** — automated sequences of steps
- Each step is a **node** that does something (read a file, post to Facebook, send an email, etc.)
- Nodes are connected together to form an automation pipeline
- **Triggers** start the workflow (a schedule, a webhook, or a manual button click)
- It runs in your browser as a web app (at `http://localhost:5678`)

**Think of it like**: If [IFTTT](https://ifttt.com) or [Zapier](https://zapier.com) had a baby with visual programming — that's n8n, but self-hosted and free!

---

## ✅ Prerequisites

Before starting, make sure you have:

1. **Docker** installed → [Install Docker](https://docs.docker.com/get-docker/)
2. **Docker Compose** installed (comes with Docker Desktop)
3. A **Facebook Page** (not a personal profile)
4. A **Facebook Developer Account** → [developers.facebook.com](https://developers.facebook.com)

---

## 🚀 Quick Start

### 1. Start n8n

Open your terminal, navigate to this folder, and run:

```bash
cd /Users/hossamyehia/me/n8n
docker-compose up -d
```

### 2. Open n8n in your browser

Go to: **http://localhost:5678**

On first launch, you'll create an admin account.

### 3. Import the workflow

See [Importing the Workflow](#importing-the-workflow) section below.

### 4. Set up Facebook credentials

See [Facebook Setup](#facebook-setup-required) section below.

### 5. Add your posts to the dataset

Edit `data/posts.json` with your content.

### 6. Activate the workflow!

Click the **Active** toggle in the top-right of the workflow editor.

---

## 🔐 Facebook Setup (Required)

This is the most important step. You need to create a Facebook App to get API access.

### Step 1: Create a Facebook App

1. Go to [Facebook Developers](https://developers.facebook.com)
2. Click **"My Apps"** → **"Create App"**
3. Select **"Business"** type
4. Fill in the app name (e.g., "My Auto Poster")
5. Click **"Create App"**

### Step 2: Add Facebook Pages API

1. In your app dashboard, click **"Add Products"**
2. Find **"Facebook Login for Business"** and click **"Set Up"**
3. Go to **Settings** → **Basic** and note your **App ID** and **App Secret**

### Step 3: Get a Page Access Token

1. Go to [Graph API Explorer](https://developers.facebook.com/tools/explorer/)
2. Select your app from the dropdown
3. Click **"Generate Access Token"**
4. Select these permissions:
   - `pages_manage_posts`
   - `pages_read_engagement`
   - `pages_show_list`
   - `publish_video` (if posting videos)
5. Click **"Generate Access Token"** and authorize
6. **Important**: Exchange this for a **long-lived token** (the short-lived one expires in 1 hour)

### Step 4: Get a Long-Lived Token

Use this URL in your browser (replace the placeholders):

```
https://graph.facebook.com/v18.0/oauth/access_token?grant_type=fb_exchange_token&client_id=YOUR_APP_ID&client_secret=YOUR_APP_SECRET&fb_exchange_token=YOUR_SHORT_LIVED_TOKEN
```

This gives you a token that lasts ~60 days.

### Step 5: Get Your Page Access Token

```
https://graph.facebook.com/v18.0/me/accounts?access_token=YOUR_LONG_LIVED_TOKEN
```

Copy the **`access_token`** from the response — this is your **Page Access Token**.
Also note the **`id`** — this is your **Page ID**.

### Step 6: Add Credentials in n8n

1. In n8n, go to **Credentials** (left sidebar)
2. Click **"Add Credential"**
3. Search for **"Facebook Graph API"**
4. Paste your **Page Access Token**
5. Click **"Save"**

---

## 🔧 How the Workflow Works

Here's what each node in the workflow does:

```
┌─────────────────────────┐
│ ⏰ Option A              │  ← Trigger: runs at 9 AM, 1 PM, 6 PM
│ Specific Times           │
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ ⏰ Option B              │  ← Trigger: runs every 4 hours
│ Periodic                 │     (Choose A or B, not both!)
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 📂 Read Posts Dataset    │  ← Reads your posts.json file
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 🔄 Get Next Unposted    │  ← Finds the next post where "posted": false
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 🖼️ Has Image?           │  ← Checks if post has an image URL
└─────┬───────────┬───────┘
      │ YES       │ NO
      ▼           ▼
┌──────────┐ ┌──────────┐
│ 📸 Post  │ │ 📝 Post  │  ← Posts to Facebook (with or without image)
│ w/ Image │ │ Text Only│
└────┬─────┘ └────┬─────┘
     │             │
     └──────┬──────┘
            ▼
┌─────────────────────────┐
│ ✅ Mark as Posted        │  ← Updates posts.json → "posted": true
└───────────┬─────────────┘
            │
            ▼
┌─────────────────────────┐
│ 🏁 Done                 │  ← Workflow complete!
└─────────────────────────┘
```

### Node Explanations:

| Node | Type | Purpose |
|------|------|---------|
| **⏰ Schedule Trigger** | Trigger | Starts the workflow on a schedule |
| **📂 Read Posts Dataset** | Read/Write File | Reads your `posts.json` file |
| **🔄 Get Next Unposted** | Code (JavaScript) | Finds the first post with `"posted": false` |
| **🖼️ Has Image?** | If/Condition | Routes to different Facebook post types |
| **📸 Post WITH Image** | Facebook Graph API | Posts content with a photo to your Page |
| **📝 Post TEXT Only** | Facebook Graph API | Posts text-only content to your Page |
| **✅ Mark as Posted** | Code (JavaScript) | Updates the JSON file to prevent re-posting |
| **🏁 Done** | No Operation | End of workflow marker |

---

## 📊 Managing Your Posts Dataset

### File Location
```
data/posts.json
```

### Format
```json
[
  {
    "id": 1,
    "text": "Your post text goes here! 🎉",
    "imageUrl": "https://your-image-url.com/image.jpg",
    "posted": false
  }
]
```

### Fields Explained

| Field | Type | Description |
|-------|------|-------------|
| `id` | Number | Unique identifier for the post (1, 2, 3...) |
| `text` | String | The text content of your Facebook post. Supports emojis! |
| `imageUrl` | String | URL to an image. Leave empty `""` for text-only posts |
| `posted` | Boolean | `false` = not yet posted, `true` = already posted |

### Tips:
- **Add new posts**: Just add new objects to the array with `"posted": false`
- **Re-post something**: Change `"posted": true` back to `"posted": false`
- **Reset everything**: Change all posts back to `"posted": false`
- **Image URLs**: Must be publicly accessible URLs (not local file paths)

---

## ⏰ Scheduling Options

### Option A: Specific Times of Day

Posts at exact hours you choose. Great for optimal engagement times!

**Default**: 9:00 AM, 1:00 PM, 6:00 PM

**How to change times:**
1. Double-click the **"⏰ Option A - Specific Times"** node
2. You'll see the list of trigger times
3. Add, remove, or modify times as needed
4. Click **"Save"**

### Option B: Every X Hours

Posts at regular intervals. Simple and consistent!

**Default**: Every 4 hours

**How to change interval:**
1. Double-click the **"⏰ Option B - Every 4 Hours"** node
2. Change the `hoursInterval` value
3. You can also change to minutes: set `field` to `"minutes"`
4. Click **"Save"**

### ⚠️ How to Choose Between Options

**IMPORTANT**: Only ONE trigger should be connected at a time!

To switch between options in n8n:
1. **Right-click** the connection line from the trigger you DON'T want
2. Click **"Delete"**
3. Make sure only your preferred trigger is connected to "📂 Read Posts Dataset"

Or you can **disable** a node by right-clicking it → **"Deactivate"**

---

## 📥 Importing the Workflow

1. Open n8n at **http://localhost:5678**
2. Click the **☰ menu** (top-left hamburger menu)
3. Click **"Import from File"**
4. Select the file: `workflows/facebook-auto-poster.json`
5. The workflow will appear in the editor!
6. **Configure the Facebook credentials** on the Facebook nodes (see [Facebook Setup](#facebook-setup-required))
7. **Activate** the workflow using the toggle in the top-right

---

## 🔄 Stopping / Restarting n8n

```bash
# Stop n8n
docker-compose down

# Start n8n
docker-compose up -d

# View logs
docker-compose logs -f n8n

# Restart n8n
docker-compose restart
```

---

## ❓ Troubleshooting

### "No more posts to publish!"
→ Add more posts to `data/posts.json` or reset `"posted"` to `false`

### Facebook API Error
→ Check that your Page Access Token is valid and not expired
→ Make sure you have the correct permissions (`pages_manage_posts`)
→ Verify your Page ID is correct

### n8n won't start
→ Make sure Docker is running
→ Check if port 5678 is already in use: `lsof -i :5678`
→ Check logs: `docker-compose logs n8n`

### Posts not appearing on my Page
→ Check the **Executions** tab in n8n to see if there were errors
→ Make sure the workflow is **Active** (toggle is ON)
→ Verify the schedule trigger is connected

### Token expired
→ Facebook tokens expire! You'll need to regenerate them periodically
→ Consider using a "System User" token for longer-lasting access

---

## 📁 Project Structure

```
n8n/
├── docker-compose.yml              # Runs n8n in Docker
├── data/
│   └── posts.json                  # Your posts dataset
├── workflows/
│   └── facebook-auto-poster.json   # The n8n workflow (import this!)
└── README.md                       # This file
```

---

## 🎓 Learn More About n8n

- [n8n Documentation](https://docs.n8n.io/)
- [n8n Community](https://community.n8n.io/)
- [n8n Workflow Templates](https://n8n.io/workflows/)
- [Facebook Graph API Docs](https://developers.facebook.com/docs/graph-api/)

---

Made with ❤️ using n8n
