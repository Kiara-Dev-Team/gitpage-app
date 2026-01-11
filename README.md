# GitPage App

A lightweight, serverless application that uses **GitHub Issues as a database** with a **Cloudflare Worker OAuth backend**. Perfect for personal projects, todo apps, blogs, or any application that needs a simple, free, and git-based data store.

## 🚀 Project Overview

GitPage App is a modern web application that leverages GitHub's infrastructure to provide a complete CRUD application without traditional database costs. The frontend is a static React app hosted on GitHub Pages, while authenticated operations are handled through a minimal Cloudflare Worker that securely manages OAuth tokens.

**Key Features:**
- 📖 **Read public issues** without authentication
- 🔐 **OAuth-based writes** for creating, updating, and closing issues
- 🗄️ **GitHub Issues as database** - leverage GitHub's robust API and UI
- ⚡ **Serverless architecture** - no servers to maintain
- 🎯 **YAML front matter** for structured data in issue bodies
- 🏷️ **Label-based filtering** for powerful querying
- 💰 **Free hosting** on GitHub Pages + Cloudflare Workers

**Architecture Flow:**
```
Frontend (GitHub Pages) → Read (public) → GitHub Issues API
                        ↓
                   Write (auth) → Cloudflare Worker → GitHub Issues API
```

---

## ✨ Features

- ✅ **Static frontend hosted on GitHub Pages** - Fast, reliable, and free
- ✅ **GitHub Issues as lightweight database** - No traditional database needed
- ✅ **Read-only public access** - Browse data without authentication
- ✅ **OAuth-protected writes** - Secure create/update/close operations
- ✅ **Minimal Cloudflare Worker** - Secure token exchange and API proxy
- ✅ **YAML front matter in issue bodies** - Structured data storage
- ✅ **Label-based filtering and indexing** - Query by type, priority, status
- ✅ **Optimistic UI updates** - Instant feedback with background sync
- ✅ **Local caching (60s TTL)** - Reduced API calls and faster loading

---

## 🛠️ Tech Stack

| Layer | Technology |
|-------|-----------|
| **Frontend** | Vite + React + TypeScript |
| **Backend** | Cloudflare Worker (TypeScript) |
| **Database** | GitHub Issues API |
| **Hosting** | GitHub Pages + Cloudflare Workers |
| **Styling** | Simple CSS/utility classes |

---

## 📋 Prerequisites

Before you begin, ensure you have:

- **Node.js 18+** and npm/pnpm/yarn
- **GitHub account** (free tier works)
- **Cloudflare account** (free tier works)
- **Git** installed
- **Wrangler CLI**: Install globally with `npm install -g wrangler`

---

## 📊 Data Model

GitHub Issues serve as individual records with structured data stored in YAML front matter:

### Example Issue Structure

**Issue Title:** `Fix authentication bug`

**Issue Body:**
```yaml
---
type: todo
id: 550e8400-e29b-41d4-a716-446655440000
createdAt: 2026-01-11T10:30:00Z
due: 2026-01-15T23:59:59Z
tags: [backend, urgent, security]
---
The OAuth flow is failing when users try to sign in from Safari.
Need to investigate cookie settings.
```

### Labels as Indexes

Labels function as database indexes for efficient filtering:

**Type Labels:**
- `type:todo`
- `type:bug`
- `type:feature`

**Priority Labels:**
- `priority:low`
- `priority:med`
- `priority:high`

**Status Labels:**
- `status:done` (or use GitHub's built-in open/closed state)

---

## 🚀 Setup Instructions

### Step 1: Clone the Repository

```bash
git clone https://github.com/Kiara-Dev-Team/gitpage-app.git
cd gitpage-app
```

### Step 2: Create a Data Repository

1. **Create a new GitHub repository** (e.g., `your-username/app-data`)
2. Make it **public** (or private if using `repo` scope)
3. **Create initial labels** using GitHub CLI:

```bash
# Using GitHub CLI (gh)
gh label create "type:todo" --repo your-username/app-data
gh label create "type:bug" --repo your-username/app-data
gh label create "type:feature" --repo your-username/app-data
gh label create "priority:low" --repo your-username/app-data
gh label create "priority:med" --repo your-username/app-data
gh label create "priority:high" --repo your-username/app-data
gh label create "status:done" --repo your-username/app-data
```

**Or create labels manually:**
- Go to your data repository on GitHub
- Navigate to **Settings** → **Labels**
- Click **New label** and create each label

### Step 3: Create GitHub OAuth App

1. Go to **GitHub Settings** → **Developer settings** → **OAuth Apps**
2. Click **New OAuth App**
3. Fill in the details:
   - **Application name**: `GitPage Issue App`
   - **Homepage URL**: `https://your-username.github.io/gitpage-app`
   - **Authorization callback URL**: `https://your-worker.your-subdomain.workers.dev/auth/callback`
     
     *(You'll get the exact Worker URL after deploying in Step 5)*
4. Click **Register application**
5. **Save the Client ID** and click **Generate a new client secret**
6. **Save the Client Secret securely** - you'll need both for the Worker

### Step 4: Configure Cloudflare Worker

#### 4.1 Install Dependencies

```bash
cd worker
npm install
```

#### 4.2 Configure wrangler.toml

Edit `worker/wrangler.toml`:

```toml
name = "gitpage-oauth-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
ALLOWED_ORIGIN = "https://your-username.github.io"
GH_DATA_OWNER = "your-username"
GH_DATA_REPO = "app-data"
```

**Important:** Replace `your-username` with your actual GitHub username.

#### 4.3 Set Secrets

```bash
cd worker
wrangler secret put GITHUB_CLIENT_ID
# Paste your OAuth App Client ID when prompted

wrangler secret put GITHUB_CLIENT_SECRET
# Paste your OAuth App Client Secret when prompted
```

#### 4.4 Deploy Worker

```bash
wrangler deploy
```

**Copy the Worker URL** from the output (e.g., `https://gitpage-oauth-worker.your-subdomain.workers.dev`)

#### 4.5 Update OAuth App Callback URL

Go back to your GitHub OAuth App settings and update:
- **Authorization callback URL**: `https://gitpage-oauth-worker.your-subdomain.workers.dev/auth/callback`

Use the exact Worker URL from the previous step.

### Step 5: Configure Frontend

#### 5.1 Install Dependencies

```bash
cd frontend
npm install
```

#### 5.2 Create Environment File

Copy the example environment file:

```bash
cp .env.example .env
```

Edit `.env` with your values:

```env
VITE_GH_OWNER=your-username
VITE_GH_REPO=app-data
VITE_WORKER_URL=https://gitpage-oauth-worker.your-subdomain.workers.dev
```

#### 5.3 Update vite.config.ts Base Path

Edit `frontend/vite.config.ts`:

```typescript
export default defineConfig({
  base: '/gitpage-app/', // Change to your repo name
  // ... rest of config
})
```

**Note:** The base path should match your repository name for GitHub Pages deployment.

### Step 6: Deploy to GitHub Pages

#### 6.1 Build Frontend

```bash
cd frontend
npm run build
```

#### 6.2 Enable GitHub Pages

1. Go to your repository on GitHub
2. Navigate to **Settings** → **Pages**
3. Under **Source**, select:
   - **Deploy from a branch**
   - **Branch**: `gh-pages` (or `main` if using root or `/docs`)
4. Click **Save**

#### 6.3 Deploy Using GitHub Actions (Recommended)

This repository includes `.github/workflows/deploy.yml` that automatically builds and deploys on push to `main`.

**Automated deployment:** Just push to `main` branch and GitHub Actions handles the rest!

**Manual deployment alternative:**

```bash
cd frontend
npm run build
npx gh-pages -d dist
```

#### 6.4 Verify Deployment

Visit your deployed app:

```
https://your-username.github.io/gitpage-app/
```

---

## 💻 Local Development

### Run Frontend Locally

```bash
cd frontend
npm run dev
# Visit http://localhost:5173
```

The frontend will hot-reload as you make changes.

### Run Worker Locally

```bash
cd worker
wrangler dev
# Worker runs on http://localhost:8787
```

### Local Testing Configuration

Update `frontend/.env` for local Worker testing:

```env
VITE_WORKER_URL=http://localhost:8787
```

**Tip:** Run both the frontend and worker in separate terminal windows for full local development.

---

## 📖 Usage

### Public Read Access

Anyone can view your data without authentication:

- **Browse all issues** from the data repository
- **Filter** by labels, state (open/closed)
- **View issue details** including YAML metadata
- **Search** through issues

### Authenticated Write Access

1. Click **"Sign in with GitHub"**
2. **Authorize** the OAuth app
3. You can now:
   - ✏️ **Create** new issues (records)
   - 📝 **Edit** existing issues (title, body, labels)
   - ✅ **Close/Reopen** issues (mark as done)

### Creating a Record

1. Click **"New Record"** button
2. Fill in the form:
   - **Title**: Brief description
   - **Description**: Detailed information
   - **Type**: todo/bug/feature
   - **Priority**: low/med/high
   - **Due date**: Optional deadline
   - **Tags**: Additional categorization
3. Click **"Create"**
4. The app creates a GitHub issue with YAML front matter

The issue will be visible in both the app and the GitHub repository UI!

---

## 🔌 API Endpoints (Worker)

The Cloudflare Worker exposes the following endpoints:

| Endpoint | Method | Description | Auth Required |
|----------|--------|-------------|---------------|
| `/auth/login` | GET | Redirects to GitHub OAuth authorization | No |
| `/auth/callback` | GET | Handles OAuth callback, exchanges code for token | No |
| `/auth/logout` | POST | Clears authentication cookie | No |
| `/api/repos/:owner/:repo/issues` | GET | Proxy to GitHub Issues API (list issues) | No* |
| `/api/repos/:owner/:repo/issues` | POST | Proxy to GitHub Issues API (create issue) | Yes |
| `/api/repos/:owner/:repo/issues/:number` | GET | Proxy to GitHub Issues API (get issue) | No* |
| `/api/repos/:owner/:repo/issues/:number` | PATCH | Proxy to GitHub Issues API (update issue) | Yes |

*_No authentication required for public repositories_

---

## 🔒 Security Features

Security is built into every layer:

- ✅ **HttpOnly cookies**: Access tokens never exposed to JavaScript
- ✅ **CORS protection**: Only allowed origin can call Worker
- ✅ **CSRF protection**: State parameter validation + custom headers
- ✅ **Path validation**: Worker only proxies allowed GitHub API paths
- ✅ **Scope limitation**: OAuth requests minimum permissions (`public_repo + read:user`)
- ✅ **No client secrets in frontend**: All sensitive data stays in Worker
- ✅ **Secure token exchange**: Authorization code flow prevents token exposure
- ✅ **Same-Site cookie attributes**: Additional CSRF protection

---

## 🔧 Troubleshooting

### Issue: "Failed to fetch issues"

**Possible causes:**
- ❌ Data repository doesn't exist or is private
- ❌ `VITE_GH_OWNER` and `VITE_GH_REPO` are incorrect in `.env`
- ❌ GitHub API rate limits exceeded (60 req/hour unauthenticated)

**Solutions:**
- ✅ Verify the data repository exists and is public
- ✅ Double-check environment variables
- ✅ Sign in to get higher rate limits (5,000 req/hour)

### Issue: OAuth redirect fails

**Possible causes:**
- ❌ OAuth App callback URL doesn't match Worker URL
- ❌ Worker `ALLOWED_ORIGIN` doesn't match GitHub Pages URL
- ❌ Worker secrets are not set correctly

**Solutions:**
- ✅ Ensure callback URL matches exactly (including protocol)
- ✅ Update `ALLOWED_ORIGIN` in `wrangler.toml`
- ✅ Re-run `wrangler secret put` commands

### Issue: "Unauthorized" when creating issues

**Possible causes:**
- ❌ Not signed in
- ❌ OAuth app doesn't have correct permissions
- ❌ Worker cookie is not being sent

**Solutions:**
- ✅ Check for user avatar in UI (indicates signed in)
- ✅ Verify OAuth app scopes include `public_repo`
- ✅ Check browser DevTools → Application → Cookies for auth cookie

### Issue: GitHub Pages shows 404

**Possible causes:**
- ❌ `base` path in `vite.config.ts` doesn't match repo name
- ❌ GitHub Pages is not enabled
- ❌ Deployment failed

**Solutions:**
- ✅ Ensure `base: '/gitpage-app/'` matches your repository name
- ✅ Verify GitHub Pages is enabled in repository settings
- ✅ Check GitHub Actions deployment logs

### Issue: CORS errors

**Possible causes:**
- ❌ `ALLOWED_ORIGIN` in Worker doesn't match exactly
- ❌ Worker is not deployed or accessible

**Solutions:**
- ✅ Ensure `ALLOWED_ORIGIN` matches exactly (no trailing slash)
- ✅ Include `https://` protocol in origin
- ✅ Redeploy worker with `wrangler deploy`

---

## 🎨 Customization

### Change Data Schema

Edit the YAML front matter structure:

1. Open `frontend/src/lib/types.ts`
2. Modify the TypeScript interfaces
3. Update the YAML parser/serializer logic

### Add More Labels

1. Add labels to your data repository:
   ```bash
   gh label create "status:in-progress" --repo your-username/app-data
   gh label create "team:backend" --repo your-username/app-data
   ```
2. Update filter UI in `frontend/src/components/Filters.tsx`

### Customize UI

**Components:** `frontend/src/components/`
- Modify React components for layout changes
- Add new components for additional features

**Styles:** `frontend/src/styles/`
- Update CSS files for visual customization
- Add Tailwind or other CSS frameworks if desired

### Change OAuth Scopes

Edit `worker/src/index.ts`:

**For private repositories:**
```typescript
const scope = 'repo read:user';
```

**For public repositories only (default):**
```typescript
const scope = 'public_repo read:user';
```

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                       GitHub Pages                          │
│                     (Static React SPA)                       │
└────────────────┬─────────────────────┬──────────────────────┘
                 │                     │
                 │ Read (public)       │ Write (authenticated)
                 │                     │
                 ▼                     ▼
        ┌─────────────────┐   ┌─────────────────────┐
        │  GitHub Issues  │   │  Cloudflare Worker  │
        │   REST API      │◄──┤   (OAuth Proxy)     │
        │  (Public Read)  │   │                     │
        └─────────────────┘   └─────────────────────┘
                 │                     │
                 │                     │
                 └──────────┬──────────┘
                            │
                            ▼
                   ┌─────────────────┐
                   │ GitHub Issues   │
                   │  (Data Repo)    │
                   │                 │
                   │ • Issues = Rows │
                   │ • Labels = Tags │
                   │ • Body = Data   │
                   └─────────────────┘
```

### Data Flow

1. **Public Read**: Frontend → GitHub API → Display data
2. **Authenticated Write**: 
   - User clicks "Sign in with GitHub"
   - Frontend → Worker `/auth/login` → GitHub OAuth
   - GitHub → Worker `/auth/callback` → Set HttpOnly cookie
   - Frontend → Worker `/api/...` (with cookie) → GitHub API

---

## 📁 Project Structure

```
gitpage-app/
├── frontend/
│   ├── src/
│   │   ├── components/        # React components
│   │   │   ├── IssueList.tsx  # Main issue list view
│   │   │   ├── IssueForm.tsx  # Create/edit form
│   │   │   ├── Filters.tsx    # Label filtering UI
│   │   │   └── Auth.tsx       # Sign in/out component
│   │   ├── lib/               # API wrappers, utilities
│   │   │   ├── github.ts      # GitHub API client
│   │   │   ├── types.ts       # TypeScript types
│   │   │   └── yaml.ts        # YAML parser/serializer
│   │   ├── styles/            # CSS files
│   │   │   └── main.css       # Main stylesheet
│   │   ├── App.tsx            # Main app component
│   │   └── main.tsx           # Entry point
│   ├── index.html
│   ├── vite.config.ts
│   ├── package.json
│   └── .env.example           # Environment template
├── worker/
│   ├── src/
│   │   └── index.ts           # Worker entry point
│   │       ├── OAuth handlers
│   │       ├── API proxy
│   │       └── CORS middleware
│   ├── wrangler.toml          # Worker configuration
│   └── package.json
├── .github/
│   └── workflows/
│       └── deploy.yml         # GitHub Actions deployment
├── LICENSE
└── README.md                  # This file
```

---

## 🤝 Contributing

We welcome contributions! Here's how to get started:

1. **Fork the repository**
   ```bash
   gh repo fork Kiara-Dev-Team/gitpage-app
   ```

2. **Create a feature branch**
   ```bash
   git checkout -b feature/amazing-feature
   ```

3. **Make your changes**
   - Write clean, documented code
   - Follow existing code style
   - Add tests if applicable

4. **Commit your changes**
   ```bash
   git commit -m 'Add amazing feature'
   ```

5. **Push to your fork**
   ```bash
   git push origin feature/amazing-feature
   ```

6. **Open a Pull Request**
   - Describe your changes
   - Link any related issues
   - Wait for review

### Development Guidelines

- Follow TypeScript best practices
- Use meaningful variable and function names
- Add comments for complex logic
- Keep components small and focused
- Test your changes locally before submitting

---

## 📄 License

This project is licensed under the **MIT License** - see the [LICENSE](LICENSE) file for details.

### MIT License Summary

- ✅ Commercial use
- ✅ Modification
- ✅ Distribution
- ✅ Private use
- ❌ Liability
- ❌ Warranty

---

## 🙏 Credits

Built with amazing open-source tools:

- [Vite](https://vitejs.dev/) - Next generation frontend tooling
- [React](https://react.dev/) - JavaScript library for building user interfaces
- [Cloudflare Workers](https://workers.cloudflare.com/) - Serverless execution environment
- [GitHub REST API](https://docs.github.com/en/rest) - GitHub's public API

**Special Thanks:**
- The GitHub team for providing such a robust API
- Cloudflare for free Worker hosting
- The open-source community for inspiration

---

## 💬 Support

Need help? Here's where to get it:

### Documentation
- 📖 [GitHub Issues API Documentation](https://docs.github.com/en/rest/issues)
- 📖 [Cloudflare Workers Documentation](https://developers.cloudflare.com/workers/)
- 📖 [GitHub OAuth Documentation](https://docs.github.com/en/developers/apps/building-oauth-apps)

### Community
- 💬 [Open an issue](https://github.com/Kiara-Dev-Team/gitpage-app/issues) in this repository
- 🐛 Report bugs with detailed reproduction steps
- 💡 Request features with use cases
- ❓ Ask questions in GitHub Discussions

### Getting Help

When asking for help, please include:
1. What you're trying to do
2. What you expected to happen
3. What actually happened
4. Relevant error messages or screenshots
5. Your configuration (without secrets!)

---

<div align="center">

**Made with ❤️ by the [Kiara Dev Team](https://github.com/Kiara-Dev-Team)**

⭐ Star this repo if you find it useful!

[Report Bug](https://github.com/Kiara-Dev-Team/gitpage-app/issues) · [Request Feature](https://github.com/Kiara-Dev-Team/gitpage-app/issues) · [Documentation](https://github.com/Kiara-Dev-Team/gitpage-app#readme)

</div>