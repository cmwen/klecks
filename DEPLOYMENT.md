# Deployment Guide

This document explains how Klecks can be deployed to various hosting environments, including GitHub Pages with sub-paths.

## GitHub Pages Deployment

Klecks is configured to work with GitHub Pages, including deployment to repository sub-paths (e.g., `https://username.github.io/repository-name/`).

### Automatic Deployment

The repository includes a GitHub Actions workflow (`.github/workflows/deploy.yml`) that automatically:
1. Builds the application
2. Creates a `.nojekyll` file (required for GitHub Pages)
3. Deploys to GitHub Pages on every push to the `main` branch

### Configuration for Sub-Path Deployment

The application uses **relative paths** for all assets, making it compatible with any URL path:

- **Build scripts** use `--public-url .` to generate relative paths
- **No `<base href>`** tag that would force absolute path resolution
- **All asset references** (CSS, JS, images, fonts) use relative paths

This means the application will work at:
- Root path: `https://example.com/`
- Sub-path: `https://example.com/klecks/`
- Any other path: `https://example.com/some/nested/path/`

### Manual Deployment to GitHub Pages

If you need to deploy manually:

```bash
# Build the application
npm ci
npm run lang:build
npm run build
npm run build:help

# Create .nojekyll file (important for GitHub Pages)
touch dist/.nojekyll

# Deploy the dist/ directory to your gh-pages branch
# (Use your preferred deployment method)
```

## Other Hosting Providers

### Static File Hosting (Netlify, Vercel, etc.)

Simply deploy the `dist/` directory after building:

```bash
npm ci
npm run lang:build
npm run build
npm run build:help
```

The `.nojekyll` file is only needed for GitHub Pages, not for other providers.

### Custom Server

You can serve the `dist/` directory with any static file server:

```bash
# Example with Python
cd dist
python3 -m http.server 8080

# Example with Node.js (using serve)
npx serve dist

# Example with nginx
# Point your nginx root to the dist/ directory
```

## Docker Deployment

See `Dockerfile` and `docker-compose.yml` for containerized deployment.

## Troubleshooting

### 404 Errors for Assets

If you see 404 errors for CSS, JS, or image files:

1. **Check that you're using the latest build** - Old builds may have absolute paths
2. **Verify the build was created with `--public-url .`** - Check `package.json` scripts
3. **Ensure no `<base href>` tag** - This was removed to support sub-path deployment
4. **For GitHub Pages: Check for `.nojekyll` file** - Should be in the root of your deployment

### Blank Page or Loading Issues

1. **Check browser console** for any error messages
2. **Verify all files were deployed** - Especially CSS and JS files
3. **Test with a local server** to isolate deployment issues

## Build Configuration

The key configuration for sub-path support is in `package.json`:

```json
{
  "scripts": {
    "build": "parcel build src/index.html --public-url .",
    "build:embed": "parcel build src/embed.ts --public-url .",
    "build:help": "parcel build src/help.html --public-url ."
  }
}
```

The `--public-url .` flag tells Parcel to use relative paths for all assets.

## Embed Mode

The embed version (`embed.js`) also uses relative paths and can be embedded in any page:

```html
<script src="path/to/dist/embed.js"></script>
<script>
  const klecks = new Klecks({
    onSubmit: (onSuccess, onError) => {
      // Handle submission
    }
  });
  // ... initialize with project
</script>
```

See `examples/embed/example.html` for a complete example.
