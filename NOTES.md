# Development Notes

## Local Development

To preview changes locally before pushing to GitHub Pages:

### Prerequisites

- Ruby and Bundler installed on your system

### Setup and Running

1. **Install dependencies**:

   ```bash
   bundle install
   ```

2. **Serve the site locally**:

   ```bash
   bundle exec jekyll serve
   ```

3. **Access your site**:
   Open your browser and go to `http://localhost:4000`

### Additional Options

- **Auto-reload on changes**:

  ```bash
  bundle exec jekyll serve --livereload
  ```

- **Include draft posts**:
  ```bash
  bundle exec jekyll serve --drafts
  ```
