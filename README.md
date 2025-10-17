# 🤖 AI-Powered Task Automation & Deployment System

An intelligent automation service that receives task descriptions, generates complete web applications using Google's Gemini AI, and automatically deploys them to GitHub Pages with zero manual intervention.

---

## 🎯 What Does This Project Do?

This is an end-to-end automated code generation and deployment pipeline that:

1. **Receives Task Requests** via REST API (`POST /ready` endpoint)
2. **Generates Complete Applications** using Gemini 2.5 Flash AI based on natural language descriptions
3. **Creates GitHub Repositories** automatically for each task
4. **Deploys to GitHub Pages** making generated apps instantly accessible online
5. **Sends Notifications** with deployment details to a callback URL

---

## 🔄 Complete Workflow

```
User submits task → API validates secret → Gemini generates code → 
Creates/updates GitHub repo → Commits & pushes → Enables GitHub Pages → 
Sends callback notification with live URL
```

---

## ✨ Key Features

- **🧠 AI-Driven Generation** - Uses Gemini 2.5 Flash for intelligent code generation
- **🔄 Round-Based Updates** - Round 1 creates new repositories, Round 2+ performs surgical updates
- **📎 Attachment Support** - Include images, mockups, or data files for AI context
- **⚡ Background Processing** - Returns HTTP 200 immediately, processes tasks asynchronously
- **🛡️ Safe Mode for Updates** - Preserves existing code structure during Round 2+ updates
- **🔁 Robust Retry Logic** - Exponential backoff for GitHub API and Gemini API calls
- **📊 Built-in Monitoring** - Health checks, status endpoints, and log viewing
- **🐳 Docker Ready** - Production-ready containerization included
- **🔒 Secret-Based Auth** - Simple but effective authentication for incoming requests

---

## 📋 How It Works (Technical Deep Dive)

### 1️⃣ Request Reception

```json
POST /ready
{
  "email": "user@example.com",
  "secret": "your-auth-secret",
  "task": "chess-game",
  "round": 1,
  "nonce": "unique-request-id",
  "brief": "Create a chess game with drag-and-drop pieces...",
  "evaluation_url": "https://webhook.site/your-id",
  "attachments": [
    {
      "name": "logo.png",
      "url": "data:image/png;base64,iVBORw0KGgo..."
    }
  ]
}
```

### 2️⃣ AI Code Generation

**Round 1 (New Project):**
- Sends task brief + attachments to Gemini 2.5 Flash
- Uses JSON schema enforcement for structured output
- AI generates `index.html`, `README.md`, and `LICENSE`
- Single-file responsive HTML app using Tailwind CSS

**Round 2+ (Surgical Updates):**
- Loads existing `index.html` from GitHub repository
- Performs minimal, targeted changes based on new brief
- Preserves core application logic and structure (Safe Mode)
- Safety checks prevent destructive rewrites

### 3️⃣ GitHub Repository Management

**Round 1:** Creates new repository via GitHub API with auto-init

**Round 2+:** Clones existing repository, updates files only

- Configures git with user credentials
- Commits with descriptive messages
- Pushes to `main` branch with force option

### 4️⃣ Deployment to GitHub Pages

- Enables GitHub Pages via API (POST or PUT based on existing config)
- Configures Pages to serve from `main` branch root (`/`)
- Retry logic with exponential backoff (5 attempts)
- Waits for Pages activation before proceeding

### 5️⃣ Callback Notification

POSTs deployment results to your `evaluation_url`:

```json
{
  "email": "user@example.com",
  "task": "chess-game",
  "round": 1,
  "nonce": "unique-request-id",
  "repo_url": "https://github.com/username/chess-game",
  "commit_sha": "abc123def456...",
  "pages_url": "https://username.github.io/chess-game"
}
```

---

## 🚀 Deployment Options

### Option 1: Docker (Recommended)

```bash
# Build the Docker image
docker build -t task-automation .

# Run the container
docker run -p 7860:7860 \
  -e GEMINI_API_KEY=your_gemini_key \
  -e GITHUB_TOKEN=your_github_token \
  -e GITHUB_USERNAME=your_username \
  -e STUDENT_SECRET=your_secret \
  task-automation
```

Access at: `http://localhost:7860`

### Option 2: Cloud Platforms

Deploy to any platform supporting Docker:

- **Hugging Face Spaces** (Free tier available, GPU optional)
- **Google Cloud Run** (Serverless, auto-scaling)
- **AWS ECS/Fargate** (Enterprise-grade)
- **Azure Container Instances** (Pay-per-use)
- **DigitalOcean App Platform** (Simple, affordable)
- **Render** (Easy deployment from GitHub)

### Option 3: Local Development

```bash
# 1. Clone the repository
git clone https://github.com/YOUR_USERNAME/TDS_Project_1.git
cd TDS_Project_1

# 2. Create virtual environment
python3 -m venv env
source env/bin/activate  # Linux/Mac
# OR
env\Scripts\activate  # Windows

# 3. Install dependencies
pip install -r requirements.txt

# 4. Configure environment variables
cp .env.example .env
# Edit .env with your API keys

# 5. Run the server
uvicorn main:app --reload --port 7860
```

Access at: `http://localhost:7860`

---

## 🔑 Required API Keys

### 1. Google Gemini API Key

1. Go to: https://aistudio.google.com/app/apikey
2. Click **"Create API Key"**
3. Copy the key (starts with `AIza...`)

**Free Tier:** 15 requests/minute, 1500 requests/day

### 2. GitHub Personal Access Token

1. Go to: **GitHub Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)**
2. Click **"Generate new token (classic)"**
3. Select scopes: `repo` (full control of private repositories)
4. Generate and copy token (starts with `ghp_...`)

⚠️ **Never commit this token to version control!**

### 3. Student Secret (Custom Auth)

- Create your own secret string (e.g., `my-secret-key-12345`)
- Used to authenticate incoming requests to `/ready` endpoint
- Can be any string you choose

---

## ⚙️ Environment Variables

Create a `.env` file in the project root:

```env
GEMINI_API_KEY=AIzaSy...your_key_here
GITHUB_TOKEN=ghp_...your_token_here
GITHUB_USERNAME=your_github_username
STUDENT_SECRET=your_custom_secret_string
```

---

## 🛠️ Technology Stack

| Component | Technology | Purpose |
|-----------|-----------|---------|
| **API Framework** | FastAPI | High-performance REST API with async support |
| **AI Model** | Gemini 2.5 Flash | Natural language to code generation |
| **Validation** | Pydantic | Request/response validation and settings |
| **Git Operations** | GitPython | Local repository management |
| **GitHub Integration** | GitHub REST API | Repository creation and Pages deployment |
| **Async Tasks** | asyncio | Background task processing |
| **HTTP Client** | httpx | Async HTTP requests with retry logic |
| **Container** | Docker | Production-ready deployment |

---

## 📁 Project Structure

```
TDS_Project_1/
├── main.py              # FastAPI application and orchestration logic
├── requirements.txt     # Python dependencies
├── Dockerfile           # Production container definition
```

---

## 📖 API Documentation

### `POST /ready`

Submit a task for AI-powered code generation and deployment.

**Request Body:**

```json
{
  "email": "user@example.com",
  "secret": "your_student_secret",
  "task": "unique-task-id",
  "round": 1,
  "nonce": "unique-request-id",
  "brief": "Detailed description of what to build...",
  "evaluation_url": "https://webhook.site/your-id",
  "attachments": [
    {
      "name": "logo.png",
      "url": "data:image/png;base64,..."
    }
  ]
}
```

**Response (200 OK):**

```json
{
  "status": "ready",
  "message": "Task unique-task-id received and processing started."
}
```

**Error Responses:**

- `401 Unauthorized` - Invalid secret
- `422 Unprocessable Entity` - Invalid request format

---

### `GET /`

Root endpoint with service status message.

**Response:**

```json
{
  "message": "Task Receiver Service running. POST /ready to submit."
}
```

---

### `GET /status`

Check service status and view last received task.

**Response:**

```json
{
  "last_received_task": {
    "task": "chess-game",
    "email": "user@example.com",
    "round": 1,
    "brief": "Create a chess game...",
    "time": "2025-10-17T10:30:00Z"
  },
  "running_background_tasks": 1
}
```

---

### `GET /health`

Health check endpoint for monitoring.

**Response:**

```json
{
  "status": "ok",
  "timestamp": "2025-10-17T10:30:00Z"
}
```

---

### `GET /logs?lines=200`

View recent application logs.

**Query Parameters:**

- `lines` (optional): Number of lines to retrieve (1-5000, default: 200)

**Response:** Plain text log output

---

## 🧪 Testing

### Test with cURL

```bash
curl -X POST http://localhost:7860/ready \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test@example.com",
    "secret": "your_student_secret",
    "task": "hello-world-test",
    "round": 1,
    "nonce": "test-001",
    "brief": "Create a simple hello world webpage with a gradient background and centered text saying Hello World!",
    "evaluation_url": "https://webhook.site/YOUR_UNIQUE_ID",
    "attachments": []
  }'
```

### Test with Postman

1. **Get a webhook URL:**
   - Go to https://webhook.site
   - Copy your unique URL

2. **Create a POST request:**
   - URL: `http://localhost:7860/ready`
   - Method: `POST`
   - Headers: `Content-Type: application/json`
   - Body: Use the JSON example above

3. **Check results:**
   - API returns immediately: `{"status": "ready", ...}`
   - Watch webhook.site for completion notification (~30-90 seconds)
   - Visit the `pages_url` in notification to see your live site

---

## 🐛 Troubleshooting

### Common Issues

**Problem:** `401 Unauthorized` response

**Solution:** Ensure the `secret` in your request matches the `STUDENT_SECRET` environment variable.

---

**Problem:** Task accepted but no notification received

**Solution:** Check application logs (`GET /logs`). Common causes:
- Invalid GitHub token or insufficient permissions
- Gemini API quota exceeded
- Invalid or unreachable `evaluation_url`
- Network connectivity issues

---

**Problem:** GitHub API errors (403, 404, 409)

**Solution:** 
- Verify GitHub token has `repo` scope
- Check if repository already exists (409 conflict in Round 1)
- Test token validity: `curl -H "Authorization: token YOUR_TOKEN" https://api.github.com/user`

---

**Problem:** Gemini AI returns invalid/incomplete responses

**Solution:** 
- Check Gemini API quota: https://aistudio.google.com/app/apikey
- Ensure API key is valid and not expired
- Review logs for specific error messages
- System has 3-4 retries with exponential backoff built-in

---

**Problem:** Round 2 updates are destructive

**Solution:** 
- Safe Mode is enabled by default
- System checks if new HTML is >70% shorter than original and reverts if so
- Review the `call_llm_round2_surgical_update` function for safety logic
- Consider making briefs more specific about preserving functionality

---

### Debug Mode

Enable detailed logging:

```bash
# Set environment variable
export LOG_LEVEL=DEBUG  # Linux/Mac
$env:LOG_LEVEL="DEBUG"  # Windows PowerShell
```

Or modify `main.py` temporarily:

```python
logger.setLevel(logging.DEBUG)
```

### Viewing Logs

**Local:**
```bash
tail -f logs/app.log
```

**Docker:**
```bash
docker logs -f CONTAINER_ID
```

**Via API:**
```bash
curl http://localhost:7860/logs?lines=500
```

---

## 📄 License

MIT License - Free to use, modify, and distribute. See LICENSE file for details.

