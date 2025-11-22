# 🚀 GitHub Pages Deployment Guide

## Overview

This guide will help you deploy your Code Playground app to GitHub Pages.

## Prerequisites

✅ Git repository initialized  
✅ GitHub account  
✅ Repository pushed to GitHub

## Deployment Steps

### 1. **Push Your Changes to GitHub**

First, commit all your recent changes:

```bash
# Add all changes
git add .

# Commit with a descriptive message
git commit -m "feat: Add cute dragon logo and improved UI"

# Push to GitHub
git push origin main
```

### 2. **Enable GitHub Pages**

1. Go to your repository on GitHub
2. Click **Settings** tab
3. Scroll down to **Pages** section (in the left sidebar)
4. Under **Source**, select:
   - Source: **GitHub Actions**
5. Click **Save**

### 3. **Automatic Deployment**

Once you push to the `main` branch:

- ✅ GitHub Actions will automatically trigger
- ✅ The workflow will build your app
- ✅ Deploy to GitHub Pages
- ✅ Your site will be live at: `https://[username].github.io/devtools-frontend-demo/`

### 4. **Monitor Deployment**

1. Go to the **Actions** tab in your repository
2. You'll see the "Deploy Demo Page" workflow running
3. Click on it to see the progress
4. Wait for both "build" and "deploy" jobs to complete (green checkmarks)

### 5. **Access Your Deployed App**

Once deployment is complete, your app will be available at:

```
https://[your-github-username].github.io/devtools-frontend-demo/
```

## Configuration Details

### GitHub Actions Workflow

Location: `.github/workflows/demo-page.yml`

**Features:**

- ✅ Triggers on push to `main` branch
- ✅ Uses Bun for fast installation
- ✅ Builds with correct base path
- ✅ Automatic deployment to GitHub Pages

**Workflow Steps:**

1. **Checkout code** - Gets your repository
2. **Setup Bun** - Installs Bun runtime
3. **Install dependencies** - Runs `bun install`
4. **Build app** - Runs `bun run build:pages`
5. **Archive artifact** - Packages the build
6. **Deploy** - Publishes to GitHub Pages

### Build Configuration

**Base Path:** `/devtools-frontend-demo/`

- Configured in `package.json` script: `build:pages`
- Ensures all assets load correctly on GitHub Pages

**Build Command:**

```bash
bun run build:pages
```

This runs: `vite build --base=/devtools-frontend-demo/`

## Troubleshooting

### Issue: Workflow Not Triggering

**Solution:**

1. Ensure you're pushing to the `main` branch
2. Check that the workflow file is in `.github/workflows/`
3. Verify the workflow is enabled in Settings > Actions

### Issue: Build Fails

**Solution:**

1. Check the Actions tab for error logs
2. Ensure all dependencies are in `package.json`
3. Test build locally: `bun run build:pages`

### Issue: 404 Page Not Found

**Solution:**

1. Verify GitHub Pages is enabled
2. Check that Source is set to "GitHub Actions"
3. Ensure base path matches repository name
4. Wait a few minutes for DNS propagation

### Issue: Assets Not Loading

**Solution:**

1. Check that `build:pages` uses correct base path
2. Verify all asset imports use relative paths
3. Clear browser cache and hard refresh

## Local Testing

Before deploying, test the production build locally:

```bash
# Build for GitHub Pages
bun run build:pages

# Preview the build
bun run preview
```

Then open: `http://localhost:4173/devtools-frontend-demo/`

## Manual Deployment (Alternative)

If you prefer manual deployment:

```bash
# Build the app
bun run build:pages

# Install gh-pages (if not installed)
bun add -D gh-pages

# Deploy to gh-pages branch
npx gh-pages -d dist
```

## Updating Your Deployment

To update your deployed app:

```bash
# Make your changes
# ...

# Commit changes
git add .
git commit -m "your message"

# Push to trigger automatic deployment
git push origin main
```

GitHub Actions will automatically rebuild and redeploy!

## Custom Domain (Optional)

To use a custom domain:

1. Add a `CNAME` file to `/public/` with your domain:

   ```
   yourdomain.com
   ```

2. Configure DNS with your domain provider:

   - Add a CNAME record pointing to `[username].github.io`

3. In GitHub Settings > Pages:
   - Enter your custom domain
   - Enable "Enforce HTTPS"

## Environment Variables

If you need environment variables:

1. Add them to GitHub Secrets:

   - Settings > Secrets and variables > Actions
   - Click "New repository secret"

2. Reference in workflow:
   ```yaml
   env:
     VITE_API_KEY: ${{ secrets.API_KEY }}
   ```

## Performance Optimization

Your build is already optimized with:

- ✅ Vite's production build
- ✅ Code splitting
- ✅ Asset optimization
- ✅ Tree shaking
- ✅ Minification

## Monitoring

### Check Deployment Status

```bash
# View recent deployments
gh api repos/:owner/:repo/pages/builds
```

### View Logs

1. Go to Actions tab
2. Click on latest workflow run
3. Expand build/deploy steps to see logs

## Security

### Permissions

The workflow has minimal permissions:

- `pages: write` - Deploy to Pages
- `id-token: write` - Verify deployment source

### Best Practices

- ✅ Don't commit secrets to repository
- ✅ Use GitHub Secrets for sensitive data
- ✅ Review workflow changes carefully
- ✅ Enable branch protection rules

## Rollback

To rollback to a previous version:

1. Go to Actions tab
2. Find the successful deployment you want to restore
3. Click "Re-run jobs"

Or manually:

```bash
# Revert to previous commit
git revert HEAD

# Push to trigger redeployment
git push origin main
```

## Deployment Checklist

Before deploying, ensure:

- ✅ All changes committed
- ✅ Build succeeds locally
- ✅ No console errors
- ✅ All features working
- ✅ Assets loading correctly
- ✅ Responsive design tested
- ✅ Cross-browser compatibility checked

## Next Steps

After successful deployment:

1. **Share your app**: `https://[username].github.io/devtools-frontend-demo/`
2. **Add README badge**:
   ```markdown
   ![Deploy Status](https://github.com/[username]/devtools-frontend-demo/actions/workflows/demo-page.yml/badge.svg)
   ```
3. **Monitor analytics** (if configured)
4. **Gather user feedback**
5. **Iterate and improve**

## Support

If you encounter issues:

- Check [GitHub Pages documentation](https://docs.github.com/en/pages)
- Review [GitHub Actions logs](https://github.com/[username]/devtools-frontend-demo/actions)
- Check [Vite deployment guide](https://vitejs.dev/guide/static-deploy.html)

## Summary

Your app is now configured for automatic deployment! 🎉

**Deployment Flow:**

```
Code Changes → Commit → Push to main → GitHub Actions → Build → Deploy → Live! 🚀
```

Every push to `main` will automatically update your live site!
