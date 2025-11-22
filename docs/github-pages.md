# Hosting Decap CMS on GitHub Pages

This guide walks through adding Decap CMS to a static site hosted on GitHub Pages.

## Prerequisites

- A public GitHub repository containing the static site you want to manage.
- GitHub Pages enabled for the repository (either from the root of `main` or from a `gh-pages` branch).
- A GitHub OAuth application and a small OAuth proxy (for example [`netlify-cms-oauth-provider`](https://github.com/netlify/netlify-cms-oauth-provider)) deployed to a service that can run Node.js or Go apps. The OAuth provider handles GitHub authentication for Decap CMS.

> GitHub Pages cannot run the OAuth proxy itself because Pages only serves static assets. Deploy the proxy elsewhere (e.g., Render, Railway, Fly.io, or Netlify Functions) and point Decap CMS to it.

## Add the CMS UI

Create an `/admin/index.html` that loads Decap CMS from the CDN. The file can live in your static site source or a `public` folder, depending on your build tool. A minimal example:

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8" />
    <title>Content Manager</title>
  </head>
  <body>
    <script src="https://unpkg.com/decap-cms@^3.0.0/dist/decap-cms.js"></script>
  </body>
</html>
```

Once deployed to GitHub Pages, the CMS lives at `https://<username>.github.io/<repo>/admin/` (or `https://<username>.github.io/admin/` for user sites).

## Configure Decap CMS

Place an `/admin/config.yml` next to the HTML file. Update the placeholders to match your repository, branch, and OAuth proxy URL.

```yaml
backend:
  name: github
  repo: <owner>/<repo>
  branch: main
  base_url: https://<your-oauth-proxy-domain>
  auth_endpoint: /auth
site_url: https://<username>.github.io/<repo>
publish_mode: editorial_workflow
media_folder: static/img/uploads
public_folder: /img/uploads
```

Key details:

- `backend.repo` and `backend.branch` must point to the repository and branch that back your GitHub Pages site.
- `backend.base_url` and `backend.auth_endpoint` should match the OAuth proxy you deployed. The proxy documentation describes the callback path to use (many default to `/auth` with a `/auth/callback` redirect handler).
- `site_url` must reflect the final GitHub Pages URL so preview links and media paths are correct.
- Adjust `media_folder`/`public_folder` to match where your static site serves assets.

## Enable GitHub authentication

1. Create a GitHub OAuth App (`Settings` → `Developer settings` → `OAuth Apps`). Set the **Homepage URL** to your GitHub Pages site and the **Authorization callback URL** to the callback path your OAuth proxy exposes (for example, `https://your-oauth-proxy.example.com/auth/callback`).
2. Configure the OAuth proxy with the client ID, client secret, and allowed repo settings, then deploy it. The proxy README will list the required environment variables.
3. Update `backend.base_url` and `backend.auth_endpoint` in `config.yml` to match the proxy deployment.

## Deploying to GitHub Pages

1. Commit the new `/admin` files and push to GitHub.
2. In **Settings → Pages**, choose the branch and folder that contain your built site (`/` for a static root, or `gh-pages` if you use a dedicated branch).
3. If your static site needs a build step, configure a GitHub Actions workflow to build to the folder GitHub Pages serves (often `./public` or `./dist`) and publish it to the Pages branch.
4. After Pages publishes, visit `/admin/` on your site. The CMS will redirect you through the OAuth proxy for GitHub authentication, then return to the admin UI where you can manage content stored in the repository.

## Troubleshooting tips

- Authentication errors usually stem from a mismatch between the OAuth App callback URL, the proxy configuration, and the `base_url`/`auth_endpoint` in `config.yml`. Confirm all three use the same domain and path.
- If the media library cannot find assets, ensure `media_folder` points to a folder committed to the repository and that `public_folder` reflects the URL where those assets are served on the published site.
- For project sites (`https://username.github.io/repo`), double-check that `site_url` includes the repository name so preview links include the right subpath.
