# Deployment Guide - Messenger Clone

## 📚 Table of Contents
1. [Prerequisites](#prerequisites)
2. [Local Development](#local-development)
3. [Build for Production](#build-for-production)
4. [Deploy to GitHub Pages](#deploy-to-github-pages)
5. [Deploy to Other Platforms](#deploy-to-other-platforms)
6. [Post-Deployment](#post-deployment)

## ✅ Prerequisites

Before deploying, ensure you have:

- ✅ Firebase project created (see FIREBASE_SETUP.md)
- ✅ Environment variables configured (.env.local)
- ✅ Security rules deployed
- ✅ Node.js 14+ and npm installed
- ✅ Git repository initialized
- ✅ All code committed

## 🔧 Local Development

### 1. Install Dependencies

```bash
npm install
```

### 2. Start Development Server

```bash
npm run dev
```

The app will open at `http://localhost:5173`

### 3. Test Features

- Create account
- Login/logout
- Send messages
- Upload images
- Check real-time sync
- Test dark/light mode

### 4. Run Linting & Formatting

```bash
npm run lint
npm run format
```

## 🏗️ Build for Production

### 1. Create Production Build

```bash
npm run build
```

This generates optimized files in `/dist` folder.

### 2. Preview Production Build

```bash
npm run preview
```

Test the production build locally at `http://localhost:4173`

### 3. Verify Build Contents

```bash
ls -la dist/
```

Should contain:
- `index.html` - Main entry point
- `assets/` - JavaScript, CSS bundles
- `manifest.json` - PWA manifest

## 🚀 Deploy to GitHub Pages

### Option 1: Using npm Script (Recommended)

```bash
npm run deploy
```

This automatically:
1. Builds the project
2. Pushes to `gh-pages` branch
3. Enables GitHub Pages deployment

### Option 2: Manual Deployment

#### Step 1: Build Project
```bash
npm run build
```

#### Step 2: Create gh-pages Branch
```bash
git checkout --orphan gh-pages
git rm -rf .
```

#### Step 3: Add Build Files
```bash
cp -r dist/* .
git add .
git commit -m "Deploy production build"
git push origin gh-pages
```

#### Step 4: Enable GitHub Pages
1. Go to repository **Settings**
2. Select **Pages** from left sidebar
3. Select **Branch**: `gh-pages`
4. Select **Folder**: `/ (root)`
5. Click **Save**

Your site will be live at: `https://Bina387.github.io/messenger-clone`

### Step 5: Update Base URL (if needed)

If deploying to a subdirectory, update `vite.config.js`:

```javascript
export default {
  base: '/messenger-clone/',
  // ... rest of config
}
```

Then rebuild and redeploy.

## 🌐 Deploy to Other Platforms

### Firebase Hosting

```bash
# Install Firebase CLI
npm install -g firebase-tools

# Login
firebase login

# Initialize
firebase init hosting

# Build
npm run build

# Deploy
firebase deploy --only hosting
```

Access at: `https://messenger-clone-PROJECT_ID.web.app`

### Netlify

```bash
# Install Netlify CLI
npm install -g netlify-cli

# Login
netlify login

# Deploy
netlify deploy --prod --dir=dist
```

Access at: `https://messenger-clone-ACCOUNT.netlify.app`

### Vercel

```bash
# Install Vercel CLI
npm install -g vercel

# Deploy
vercel --prod
```

Access at: `https://messenger-clone.ACCOUNT.vercel.app`

### AWS S3 + CloudFront

```bash
# Build
npm run build

# Sync to S3
aws s3 sync dist/ s3://your-bucket-name

# Invalidate CloudFront cache
aws cloudfront create-invalidation --distribution-id YOUR_DIST_ID --paths "/*"
```

### Docker Deployment

Create `Dockerfile`:

```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

EXPOSE 3000

ENV NODE_ENV=production
CMD ["npm", "run", "preview"]
```

Build and run:

```bash
docker build -t messenger-clone .
docker run -p 3000:3000 messenger-clone
```

## ✨ Post-Deployment

### 1. Verify Deployment

- [ ] Visit your deployed URL
- [ ] Test login/signup
- [ ] Send a test message
- [ ] Check file uploads work
- [ ] Test dark/light mode
- [ ] Verify responsive design

### 2. Set Up Custom Domain (Optional)

#### GitHub Pages

1. Go to **Settings** → **Pages**
2. Enter custom domain under "Custom domain"
3. Add DNS record to your domain:
   - Type: CNAME
   - Name: www (or @)
   - Value: `Bina387.github.io`
4. Wait for DNS propagation (5-48 hours)
5. Check "Enforce HTTPS"

#### Firebase Hosting

```bash
firebase hosting:domain:create
```

#### Netlify

1. Site settings → Domain management
2. Add custom domain
3. Update DNS records as shown

### 3. Set Up SSL/HTTPS

- ✅ GitHub Pages: Automatic
- ✅ Firebase Hosting: Automatic
- ✅ Netlify: Automatic via Let's Encrypt
- ✅ Vercel: Automatic
- 🔧 AWS S3: Use CloudFront + ACM certificate

### 4. Monitor Performance

#### Lighthouse Audit
```bash
# Run Lighthouse locally
npm install -g lighthouse
lighthouse https://your-deployed-url --view
```

#### Firebase Analytics
1. Go to Firebase Console
2. Enable Google Analytics
3. Track user events
4. Monitor performance metrics

#### Network Monitoring
- Check DevTools Network tab
- Monitor bandwidth usage
- Optimize large assets

### 5. Enable Security Headers

Add to deployment:

```
# GitHub Pages: .github/workflows/deploy.yml
# Firebase: firebase.json
# Netlify: netlify.toml
# Vercel: vercel.json

{
  "headers": [
    {
      "source": "/**",
      "headers": [
        {
          "key": "Content-Security-Policy",
          "value": "default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.firebase.com; style-src 'self' 'unsafe-inline';"
        },
        {
          "key": "X-Frame-Options",
          "value": "SAMEORIGIN"
        },
        {
          "key": "X-Content-Type-Options",
          "value": "nosniff"
        },
        {
          "key": "Referrer-Policy",
          "value": "strict-origin-when-cross-origin"
        }
      ]
    }
  ]
}
```

### 6. Set Up CI/CD Pipeline

#### GitHub Actions Example

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - run: npm install
      - run: npm run build
      
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

### 7. Environment Variables

Update deployed environment:

**GitHub Pages**: Cannot use env files directly
- Store sensitive data in GitHub Secrets
- Pass as build variables

**Firebase Hosting**: Create `.env.production`

```bash
VITE_FIREBASE_API_KEY=your_production_key
# ... other vars
```

**Netlify**: Add in Site settings → Build & deploy → Environment

**Vercel**: Add in Project settings → Environment Variables

## 🔄 Continuous Deployment Workflow

### Recommended Git Flow

```bash
# Create feature branch
git checkout -b feature/new-feature

# Make changes
git add .
git commit -m "Add new feature"

# Push to GitHub
git push origin feature/new-feature

# Create Pull Request on GitHub

# After review, merge to main
git checkout main
git merge feature/new-feature
git push origin main

# Deployment triggers automatically!
```

## 📊 Deployment Checklist

- [ ] All code committed and pushed
- [ ] `.env.local` has all required variables
- [ ] `npm run build` completes without errors
- [ ] `npm run preview` works locally
- [ ] All tests pass (if applicable)
- [ ] Security rules deployed
- [ ] Custom domain configured (if applicable)
- [ ] SSL certificate valid
- [ ] Analytics configured
- [ ] Backup enabled
- [ ] Error tracking enabled (optional)
- [ ] Performance monitoring enabled

## 🆘 Troubleshooting Deployment

### Build Fails

```bash
# Clear cache and reinstall
rm -rf node_modules dist
npm install
npm run build
```

### GitHub Pages Not Updating

```bash
# Force push to gh-pages
git push origin --force-with-lease gh-pages
```

### Custom Domain Not Working

- Verify DNS records propagated: `nslookup yourdomain.com`
- Check DNS configuration in domain provider
- Wait 24-48 hours for full propagation
- Clear browser cache

### Firebase Auth Not Working

- Check API key in Firebase Console
- Verify authentication enabled
- Check security rules allow operations
- Test with Firefox (rule out browser cache)

### Images Not Loading

- Verify Storage rules deployed
- Check file URLs in browser console
- Verify CORS settings if cross-domain
- Check Firebase Storage has files

### Environment Variables Not Loading

- Verify `.env.local` exists
- Confirm variable names start with `VITE_`
- Restart dev server
- Check `.gitignore` doesn't exclude .env

## 📈 Performance Optimization

After deployment:

1. **Enable Compression**
   - Gzip/Brotli enabled
   - Assets < 1MB ideally

2. **Optimize Images**
   - Use WebP format
   - Lazy load images
   - Resize large images

3. **Cache Strategy**
   - Service Workers for PWA
   - Browser cache headers
   - CDN caching

4. **Code Splitting**
   - Lazy route loading
   - Dynamic imports
   - Minimize main bundle

## 📞 Support

For deployment issues:
- Check [GitHub Actions logs](https://github.com/Bina387/messenger-clone/actions)
- Review Firebase Dashboard for errors
- Check browser DevTools Console
- See [Troubleshooting Guide](#troubleshooting-deployment)

---

**Last Updated**: May 2026  
**Deployment Ready**: ✅ Production
