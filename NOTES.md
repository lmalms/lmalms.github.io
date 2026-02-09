# Development Notes

## Local Development

To preview changes locally before pushing to GitHub Pages:

### Prerequisites

You need **Ruby** (2.6 or newer) and **Bundler** on your system.

#### Installing Ruby and Bundler using Homebrew

```bash
brew install ruby
```

Add Homebrew’s Ruby to your PATH (Homebrew will print the exact line after install), e.g.:

```bash
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

After installing Ruby, confirm versions:

```bash
ruby --version
bundle --version
```

### Setup and Running

1. **Install dependencies**:

   ```bash
   bundle install --path vendor/bundle
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
