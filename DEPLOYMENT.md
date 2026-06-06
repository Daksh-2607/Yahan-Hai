# Deployment Guide - YahanHai

This guide covers two deployment strategies for the YahanHai application.

---

## Option 1: Vercel (Simplest - Full Stack on Vercel)

**Pros**: Single platform, easy to manage, free tier available  
**Cons**: WebSockets limited to 30s, file uploads not persistent

### Prerequisites
- Vercel account: https://vercel.com
- MongoDB Atlas account: https://cloud.mongodb.com
- Git pushed to GitHub

### Step 1: Set up MongoDB Atlas

1. Go to https://www.mongodb.com/cloud/atlas
2. Sign up or log in
3. Create a new project and cluster (free tier)
4. Click **Database Access** → Add new database user
   - Username: `yahanhai_user`
   - Password: `(generate strong password, copy it)`
5. Click **Network Access** → Add IP Address
   - Add `0.0.0.0/0` (or your IP for production)
6. Click **Connect** → **Connect your application**
7. Copy the connection string and replace:
   - `<password>`: Your database password
   - `<dbname>`: `yahanhai`
   - Example: `mongodb+srv://yahanhai_user:password123@cluster0.xxxxx.mongodb.net/yahanhai?retryWrites=true&w=majority`

### Step 2: Deploy Backend to Vercel

```bash
# Install Vercel CLI globally
npm install -g vercel

# Navigate to backend directory
cd backend

# Login to Vercel
vercel login

# Deploy backend
vercel --prod
```

When prompted:
- **Project name**: `yahanhai-backend` (or your choice)
- **Link to existing project**: No (new project)
- **Which directory**: `.` (current directory)
- **Override settings**: No

After deployment, Vercel will show your backend URL: `https://yahanhai-backend-xxx.vercel.app`

### Step 3: Add Environment Variables to Backend (Vercel Dashboard)

1. Go to https://vercel.com/dashboard
2. Select your `yahanhai-backend` project
3. Go to **Settings** → **Environment Variables**
4. Add these variables:

| Key | Value |
|-----|-------|
| `MONGO_URI` | `mongodb+srv://yahanhai_user:password@cluster0.xxxxx.mongodb.net/yahanhai?retryWrites=true&w=majority` |
| `PORT` | `3000` |
| `JWT_SECRET` | Generate a random string (e.g., `your_super_secret_key_12345`) |
| `CLIENT_URL` | `https://yahanhai-xxx.vercel.app` (your frontend URL, add after frontend deploy) |

5. Click **Save**
6. Redeploy backend: In project → **Deployments** → Select latest → **Redeploy**

### Step 4: Deploy Frontend to Vercel

```bash
# Navigate to frontend directory
cd ../frontend

# Create .env.local with backend URL
# On Windows PowerShell:
echo "REACT_APP_API_URL=https://yahanhai-backend-xxx.vercel.app/api" | Out-File .env.local -Encoding UTF8

# Deploy
vercel --prod
```

When prompted:
- **Project name**: `yahanhai-frontend` (or your choice)
- **Link to existing project**: No (new project)

After deployment, you'll get: `https://yahanhai-xxx.vercel.app`

### Step 5: Update Backend CLIENT_URL

1. Go back to https://vercel.com/dashboard
2. Select `yahanhai-backend` project
3. **Settings** → **Environment Variables**
4. Update `CLIENT_URL` to your frontend URL: `https://yahanhai-xxx.vercel.app`
5. **Redeploy** the backend

### ✅ Deployment Complete
- Frontend: `https://yahanhai-xxx.vercel.app`
- Backend API: `https://yahanhai-backend-xxx.vercel.app`

---

## Option 2: Vercel Frontend + Railway Backend (Recommended)

**Pros**: Full WebSocket support, no 30s timeout, better for real-time features  
**Cons**: Two platforms to manage

### Prerequisites
- Vercel account: https://vercel.com
- Railway account: https://railway.app
- MongoDB Atlas account: https://cloud.mongodb.com

### Step 1: MongoDB Atlas (Same as Option 1)
Follow **Option 1, Step 1** above.

### Step 2: Deploy Backend to Railway

1. Go to https://railway.app and sign up with GitHub
2. Click **New Project** → **Deploy from GitHub repo**
3. Select your `Yahan-Hai` repository
4. Select the `backend` directory in the Service configuration

**Configure Environment Variables**:

Click the backend service → **Variables** tab and add:

| Key | Value |
|-----|-------|
| `MONGO_URI` | `mongodb+srv://yahanhai_user:password@cluster0.xxxxx.mongodb.net/yahanhai?retryWrites=true&w=majority` |
| `PORT` | `5000` |
| `JWT_SECRET` | Your secret key |
| `CLIENT_URL` | `https://yahanhai-xxx.vercel.app` (after frontend deploy) |
| `NODE_ENV` | `production` |

**Generate Public URL**:

1. Click the backend service
2. Go to **Settings** tab
3. Scroll to **Networking** → Enable **Public Networking**
4. Copy the generated Railway URL (e.g., `https://yahanhai-backend-production.railway.app`)

### Step 3: Deploy Frontend to Vercel

```bash
cd frontend

# Create .env.local with Railway backend URL
echo "REACT_APP_API_URL=https://yahanhai-backend-production.railway.app/api" | Out-File .env.local -Encoding UTF8

# Deploy to Vercel
vercel --prod
```

Get your frontend URL: `https://yahanhai-xxx.vercel.app`

### Step 4: Update Backend CLIENT_URL on Railway

1. Go to Railway dashboard
2. Select your backend service
3. **Variables** tab
4. Update `CLIENT_URL` to your Vercel frontend URL
5. Railway auto-redeploys on variable change ✅

### ✅ Deployment Complete
- Frontend (Vercel): `https://yahanhai-xxx.vercel.app`
- Backend (Railway): `https://yahanhai-backend-production.railway.app`
- Database (MongoDB Atlas): Connected

---

## Comparison

| Feature | Vercel | Railway |
|---------|--------|---------|
| Frontend | ✅ Perfect | ✅ Works |
| Backend | ⚠️ Limited | ✅ Perfect |
| WebSockets | ⚠️ 30s timeout | ✅ Full support |
| File Uploads | ❌ Not persistent | ✅ Persistent volumes available |
| Real-time features | ⚠️ May disconnect | ✅ Stable |
| Cost | Free tier available | Free tier with credits |
| Setup complexity | Easy | Easy |

**Recommendation**: Use **Option 2** (Vercel Frontend + Railway Backend) for production because your app heavily relies on WebSockets and real-time features.

---

## Troubleshooting

### Backend not connecting to MongoDB
- Check `MONGO_URI` is correct
- Verify IP whitelist in MongoDB Atlas includes Vercel/Railway IPs
- For Vercel/Railway, add `0.0.0.0/0` temporarily for testing

### Frontend API errors
- Verify `REACT_APP_API_URL` is set correctly
- Check backend `CLIENT_URL` matches frontend domain
- Check CORS is enabled in backend

### WebSocket disconnections (Vercel)
- This is expected on Vercel free tier (30s limit)
- **Solution**: Use Railway for backend (Option 2)

### Files not persisting
- Vercel: Use AWS S3 or Cloudinary for file storage
- Railway: Can configure persistent volumes

---

## Environment Variables Reference

### Backend Required
```
MONGO_URI=mongodb+srv://user:pass@cluster.mongodb.net/yahanhai
PORT=5000 (or 3000 for Vercel)
JWT_SECRET=your_secret_key
CLIENT_URL=https://frontend-url.vercel.app
NODE_ENV=production
```

### Frontend Required
```
REACT_APP_API_URL=https://backend-url/api
```

---

## Pushing Config to GitHub

```bash
# Stage all deployment files
git add frontend/vercel.json backend/vercel.json backend/.vercelignore

# Commit
git commit -m "Add deployment configuration for Vercel and Railway"

# Push
git push origin main
```

---

## Quick Links

- MongoDB Atlas: https://cloud.mongodb.com
- Vercel Dashboard: https://vercel.com/dashboard
- Railway Dashboard: https://railway.app/dashboard
- GitHub Repo: https://github.com/Daksh-2607/Yahan-Hai
