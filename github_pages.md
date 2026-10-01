# GitHub Pages deployment

This site uses the custom Jekyll build in `.github/workflows/jekyll.yml`.
GitHub Actions installs the Ruby version from `.ruby-version` and dependencies
from `Gemfile.lock`, builds the site on Ubuntu 24.04, and publishes the `_site`
artifact through GitHub Pages.

In the repository's **Settings → Pages**, choose **GitHub Actions** as the
deployment source. A push to `master` or a manual workflow run builds and
deploys the site. Pull requests targeting `master` build without deploying.

The Moonwalk layouts, includes, and assets come from this repository. No remote
theme is fetched during the build. SEO metadata uses `jekyll-seo-tag`.

The production URL and path are configured in `_config.yml`; the build uses
the base path returned by GitHub Pages. Docker is only used for local editing
and is not needed by the deployment workflow.

For local preview and dependency upgrades, see [README.md](README.md).
