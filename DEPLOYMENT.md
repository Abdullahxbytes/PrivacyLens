# PrivacyLens Deployment Guide

## Render Deployment

This guide covers deploying PrivacyLens to [Render](https://render.com). PrivacyLens consists of two services:
- **Backend**: Node.js/Express API (port 4000)
- **Frontend**: Next.js application (port 3000)

### Prerequisites

1. A [Render account](https://render.com) (free tier available)
2. A [Groq API key](https://console.groq.com/keys) for AI analysis
3. GitHub repository connected to your Render account

### Option 1: Using Render Blueprint (Recommended)

The easiest way to deploy is using the `render.yaml` configuration file.

#### Steps:

1. **Push your code to GitHub**
   ```bash
   git add render.yaml backend/Dockerfile frontend/Dockerfile
   git commit -m "Add Render deployment configuration"
   git push origin main
   ```

2. **Create a new Blueprint on Render**
   - Go to [Render Dashboard](https://dashboard.render.com)
   - Click "New +" → "Blueprint"
   - Select your GitHub repository
   - Render will automatically detect the `render.yaml` file
   - Click "Apply"

3. **Configure Environment Variables**
   
   After the services are created:
   
   **For Backend Service (privacylens-backend):**
   - Go to the backend service dashboard
   - Navigate to "Environment" tab
   - Add the following environment variables:
     ```
     PORT = 4000
     GROQ_API_KEY = your_groq_api_key_here
     GROQ_MODEL = openai/gpt-oss-120b
     ```
   
   **For Frontend Service (privacylens-frontend):**
   - Go to the frontend service dashboard
   - Navigate to "Environment" tab
   - Add the following environment variable:
     ```
     NEXT_PUBLIC_API_URL = https://<your-backend-service-name>.onrender.com
     ```
     Replace `<your-backend-service-name>` with the actual URL of your backend service (visible in the backend service dashboard)

4. **Deploy**
   - Both services will automatically deploy
   - The frontend will be available at `https://<your-frontend-service-name>.onrender.com`

5. **Update Frontend with Backend URL**
   - After the backend service is live, copy its URL
   - Update the frontend's `NEXT_PUBLIC_API_URL` environment variable
   - Redeploy the frontend service

### Option 2: Manual Service Creation

If you prefer to create services manually:

#### Backend Service

1. **Create Web Service**
   - Name: `privacylens-backend`
   - Type: Web Service
   - Environment: Node
   - Region: Choose closest to your users
   - Branch: `main` or your deployment branch
   - Root Directory: `backend`
   - Build Command: `npm install && npm run build`
   - Start Command: `npm start`
   - Port: 4000

2. **Add Environment Variables**
   ```
   PORT = 4000
   GROQ_API_KEY = your_groq_api_key_here
   GROQ_MODEL = openai/gpt-oss-120b
   ```

3. **Create Service**

#### Frontend Service

1. **Create Web Service**
   - Name: `privacylens-frontend`
   - Type: Web Service
   - Environment: Node
   - Region: Same as backend
   - Branch: `main` or your deployment branch
   - Root Directory: `frontend`
   - Build Command: `npm install && npm run build`
   - Start Command: `npm start`
   - Port: 3000

2. **Add Environment Variables**
   ```
   PORT = 3000
   NEXT_PUBLIC_API_URL = https://<backend-service-url>.onrender.com
   ```

3. **Create Service**

### Option 3: Docker Deployment

If you prefer Docker containers:

#### Backend

1. **Create Web Service**
   - Name: `privacylens-backend`
   - Type: Web Service
   - Environment: Docker
   - Dockerfile Path: `backend/Dockerfile`
   - Docker Context: `backend`
   - Port: 4000

2. **Add Environment Variables** (same as above)

#### Frontend

1. **Create Web Service**
   - Name: `privacylens-frontend`
   - Type: Web Service
   - Environment: Docker
   - Dockerfile Path: `frontend/Dockerfile`
   - Docker Context: `frontend`
   - Port: 3000

2. **Add Environment Variables** (same as above)

### Configuration Files

The repository includes:

- `render.yaml` - Render Blueprint configuration for both services
- `backend/Dockerfile` - Docker configuration for backend
- `frontend/Dockerfile` - Docker configuration for frontend

### Environment Variables

#### Backend Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `PORT` | Server port | `4000` |
| `GROQ_API_KEY` | Groq API key for AI analysis | `your_api_key_here` |
| `GROQ_MODEL` | Groq model identifier | `openai/gpt-oss-120b` |

#### Frontend Required Variables

| Variable | Description | Example |
|----------|-------------|---------|
| `NEXT_PUBLIC_API_URL` | Backend API URL | `https://privacylens-backend.onrender.com` |
| `PORT` | Next.js server port | `3000` |

### Post-Deployment Checks

1. **Health Endpoints**
   - Backend: `https://<backend-url>.onrender.com/health`
   - Frontend: `https://<frontend-url>.onrender.com`

2. **API Test**
   ```bash
   curl "https://<backend-url>.onrender.com/api/search?q=spotify"
   ```

3. **Frontend Test**
   - Open the frontend URL in a browser
   - Search for an app (e.g., "Spotify")
   - Verify the search results appear
   - Select an app and verify the analysis works

### Troubleshooting

#### Common Issues

1. **Backend fails to start**
   - Check logs in Render dashboard
   - Ensure all environment variables are set
   - Verify Groq API key is valid

2. **Frontend can't connect to backend**
   - Verify `NEXT_PUBLIC_API_URL` is correct
   - Check if CORS is properly configured (backend should allow frontend origin)
   - Ensure backend service is running

3. **Build fails**
   - Check Node.js version compatibility
   - Run `npm install` locally to verify dependencies
   - Check for missing environment variables during build

4. **Slow cold starts**
   - Render free tier has cold starts
   - Consider upgrading to a paid plan for better performance
   - Use Render's "Always On" feature for critical services

#### Logs and Monitoring

- View logs: Render Dashboard → Service → Logs
- Check environment variables: Render Dashboard → Service → Environment
- Monitor health: Use the health endpoints

### Cost Considerations

- **Free Tier**: Sufficient for development and light usage
- **Groq API**: Pay-per-use (check [Groq pricing](https://groq.com/pricing))
- **Render**: Free tier includes:
  - 1 web service
  - 1 background worker
  - 512 MB RAM
  - Limited CPU

For production use, consider:
- Upgrading Render services
- Monitoring Groq API usage
- Setting up budget alerts

### Custom Domain (Optional)

To use a custom domain:

1. Go to Render Dashboard → Settings → Custom Domains
2. Add your domain
3. Configure DNS records with your registrar
4. For frontend, you may need to:
   - Add the domain to Next.js config
   - Update CORS settings in backend

### Security Considerations

1. **API Keys**
   - Never commit API keys to git
   - Use Render's environment variable system
   - Rotate keys if compromised

2. **CORS**
   - Backend should validate origin headers
   - Only allow known frontend domains

3. **Rate Limiting**
   - Consider adding rate limiting to backend API
   - Groq API has its own rate limits

4. **HTTPS**
   - Render provides HTTPS by default
   - No additional configuration needed

### Performance Optimization

1. **Caching**
   - Backend uses in-memory cache for retrieval
   - Consider adding Redis for persistent caching

2. **CDN**
   - Frontend static assets can be served via CDN
   - Render has built-in CDN for static sites

3. **Scaling**
   - Monitor service metrics
   - Scale services based on traffic
   - Consider auto-scaling for variable load

### Updating Deployment

1. **Code Changes**
   ```bash
   git add .
   git commit -m "Your changes"
   git push origin main
   ```
   - Render auto-deploys on push (if configured)

2. **Environment Variables**
   - Update in Render dashboard
   - Redeploy service to apply changes

3. **Dependencies**
   - Update `package.json`
   - Push changes to trigger rebuild

### Destroying Deployment

To remove all services:
1. Go to Render Dashboard
2. Delete each service individually
3. Remove custom domains if configured

### Additional Resources

- [Render Documentation](https://render.com/docs)
- [Next.js Deployment Guide](https://nextjs.org/docs/deployment)
- [Express Deployment Guide](https://expressjs.com/en/advanced/best-practice-performance.html)
- [Groq API Documentation](https://console.groq.com/docs)

---

## Alternative Deployment Options

If you prefer not to use Render, here are alternatives:

### Vercel (Frontend + Serverless Functions)
- Frontend: Deploy as Next.js app
- Backend: Use Vercel Serverless Functions or separate deployment

### AWS
- Frontend: S3 + CloudFront or Amplify
- Backend: EC2, ECS, or Lambda

### Google Cloud
- Frontend: Cloud Run or App Engine
- Backend: Cloud Run or App Engine

### Fly.io
- Both services as Docker containers
- Good for global deployment

### Heroku
- Both services as separate apps
- Free tier available (with limitations)

---

**Note**: The Render free tier may have limitations. For production use with consistent performance, consider upgrading to a paid plan or using a different hosting provider that better fits your needs.
