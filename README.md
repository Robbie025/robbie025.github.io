# Levelling Automation

Varun Gopinath's personal blog on robotics, automation, and safety.

Published at [robbie025.github.io](https://robbie025.github.io).

## Preview on Ubuntu 24.04

Install [Docker Engine and the Compose plugin](https://docs.docker.com/engine/install/ubuntu/).
Check that `docker info` and `docker compose version` work. If Docker reports a
socket permission error, follow Docker's [Linux post-installation instructions](https://docs.docker.com/engine/install/linux-postinstall/)
or run the Compose commands with `sudo`.

From this repository directory, start the preview:

```sh
docker compose up --build
```

Open **http://localhost:4000**. Edit posts in `_posts`, pages, layouts, or styles
with your usual editor. The mounted files are watched and the browser reloads
automatically. The first build downloads Ruby dependencies; subsequent builds
reuse the dependency layer until the dependency files change.

Stop with Ctrl+C, then remove the stopped container with:

```sh
docker compose down
```

The preview uses Ruby 3.4.11 and Jekyll 4.4. Docker contains all Ruby tools, so
you do not need Ruby or Bundler installed on Ubuntu. Ports 4000 and 35729 are
bound to localhost. Build output and caches stay inside the container under
`/tmp`, and `_config.local.yml` supplies the local URL. Production uses
`_config.yml`.

The container defaults to user/group 1000, which is Ubuntu's usual first user.
If your account uses different IDs, pass them when starting or running it:

```sh
LOCAL_UID=$(id -u) LOCAL_GID=$(id -g) docker compose up --build
```

To preview unpublished drafts, override the normal command:

```sh
docker compose run --rm --service-ports site bundle exec jekyll serve \
  --config _config.yml,_config.local.yml --host 0.0.0.0 --livereload \
  --force_polling --drafts --destination /tmp/jekyll-site
```

Stop the normal preview before using `--service-ports` so the ports are free.

## Build the published site locally

```sh
docker compose run --rm -e JEKYLL_ENV=production site bundle exec jekyll build \
  --destination /tmp/jekyll-production
```

The temporary build output is removed with the container.

## Upgrade dependencies

The committed `Gemfile.lock` fixes gem versions for both Docker and GitHub
Actions. Docker normally installs with Bundler's frozen mode so previews
cannot silently change those versions.

1. Review upstream changes and update constraints in `moonwalk.gemspec` when
   moving to a new dependency series.
2. Regenerate the lockfile in a disposable container. A temporary gem path
   lets the container install updates as your user:

   ```sh
   docker compose run --rm -e BUNDLE_FROZEN=false \
     -e BUNDLE_PATH=/tmp/bundle-update site bundle update --all
   ```

   To upgrade Bundler, run the same container command with
   `bundle update --bundler VERSION` before updating the other gems.
3. Run `docker compose build --pull`, restart the preview, and run the
   production build. Check the homepage, blog, posts, styles, and `/feed.xml`.
4. Commit `Gemfile.lock` with the dependency changes. For a Ruby upgrade,
   change `.ruby-version` and the Dockerfile image tag together. The image
   build checks that they match; GitHub Actions reads `.ruby-version`.

The theme's source is maintained in this repository. Jekyll supplies the
compatible Rouge syntax highlighter, and `jekyll-seo-tag` supplies SEO metadata.

## GitHub Pages

Set **Settings → Pages → Build and deployment → Source** to **GitHub Actions**.
The workflow in `.github/workflows/jekyll.yml` builds on Ubuntu 24.04 using the
committed Ruby version and lockfile. Pull requests to `master` run a build;
pushes to `master` and manual workflow runs also deploy the generated `_site`
artifact to Pages.

The Docker files are for local editing and are excluded from the published
site. See [github_pages.md](github_pages.md) for deployment details.
