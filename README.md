I build this webpage
using [al-folio](https://github.com/alshedivat/al-folio)
-- a simple, clean, and responsive Jekyll theme for academics.
To find out how it works, look up its [README](https://github.com/alshedivat/al-folio/blob/master/README.md).

## Publishing a new blog post

1. **Branch**

   ```bash
   git checkout master
   git pull
   git checkout -b post/<short-slug>
   ```

2. **Add the post**

   Create `_posts/YYYY-MM-DD-<short-slug>.md` with front matter like:

   ```yaml
   ---
   layout: post
   title: Your title
   date: YYYY-MM-DD
   description: "One sentence for the post list and link previews."
   tags: space separated tags
   categories: ai
   thumbnail: assets/img/YYYY-MM-DD-<short-slug>/some-image.jpg
   giscus_comments: false
   related_posts: false
   ---
   ```

   `giscus_comments: false` turns comments off for just this post (giscus itself
   is not configured in `_config.yml`, so leaving this unset — or `true` — shows
   a "giscus comments misconfigured" error box on the live page).

3. **Add images**

   Put every image for the post in its own folder:
   `assets/img/YYYY-MM-DD-<short-slug>/`

   Reference them with the `figure.liquid` include, not plain Markdown
   `![]()`:

   ```liquid
   {% include figure.liquid path="assets/img/YYYY-MM-DD-<short-slug>/name.png" class="img-fluid rounded z-depth-1" zoomable=true alt="..." caption="..." %}
   ```

   For a single smaller image, add `width="280"` (or similar) to the include
   instead of CSS — `figure.liquid` ignores a plain `style="max-width:..."`
   attribute. For several images side by side, wrap them in Bootstrap's grid:

   ```html
   <div class="row mt-3">
     <div class="col-sm mt-3 mt-md-0">{% include figure.liquid ... %}</div>
     <div class="col-sm mt-3 mt-md-0">{% include figure.liquid ... %}</div>
   </div>
   ```

4. **Preview locally before touching master**

   ```bash
   docker compose up
   ```

   Then open `http://localhost:4000` (or whatever port `docker-compose.yml`
   maps) and check the new post. Stop it with `Ctrl+C`.

   Don't run `docker compose up` again right before committing a
   `Gemfile.lock` fix — a non-frozen local `bundle install` can silently
   rewrite `Gemfile.lock`'s `CHECKSUMS` section back to a broken state. If
   you've just fixed `Gemfile.lock`, commit and push it immediately instead.

5. **Commit, merge, push**

   ```bash
   git add _posts/YYYY-MM-DD-<short-slug>.md assets/img/YYYY-MM-DD-<short-slug>/
   git commit -m "Add post: <title>"
   git checkout master
   git merge post/<short-slug>
   git push
   ```

6. **Check CI**

   GitHub Actions deploys the site on every push to `master`
   (`.github/workflows/deploy.yml`). Check
   <https://github.com/DonSchott/donschott.github.io/actions> for a green
   check. The Prettier formatting check is cosmetic and can be ignored if it
   fails; "Deploy site" and the broken-links checks should be green.

   The site goes live at <https://benjaminschott.com> a minute or two after
   a green "Deploy site" run, built onto the `gh-pages` branch automatically
   — never push to `gh-pages` by hand.
