# Backend Architect Portfolio

A Hugo-powered technical portfolio website with a custom "technical-industrial" design aesthetic. Features a fixed sidebar navigation, warm parchment content area, and sophisticated typography using Inter and Space Grotesk fonts.

## Design System

Based on the specifications in `DESIGN.md`:

- **Colors**: Deep earthy black sidebar (#202219), warm parchment surface (#f0f4f3)
- **Typography**: Inter for body text, Space Grotesk for labels and metadata
- **Layout**: Fixed 280px sidebar with 1100px max-width content area
- **Style**: Technical-industrial aesthetic with hairline borders and ledger-like rows

## Structure

```
github_io_website/
├── content/
│   ├── blogs/          # 6 technical blog posts
│   ├── projects/         # 3 project case studies
│   ├── resume/           # Professional experience
│   ├── contact/          # Contact form
│   └── _index.md         # Homepage
├── themes/
│   └── architect/        # Custom theme
│       ├── layouts/      # HTML templates
│       └── ...
└── hugo.toml             # Site configuration
```

## Pages

1. **Home** - Hero section with latest blogs and expertise cards
2. **Blogs** - Technical journal with 6 articles:
   - Architecting for Zero-Downtime Data Migrations
   - Low-Latency Garbage Collection in Modern Java
   - The Cost of Consistency: CAP Theorem Revisited
   - Memory-Efficient Serialization with Protobuf
   - Async Rust: Zero-Cost Abstractions in Production
   - CockroachDB: Distributed SQL at Scale

3. **Projects** - 3 engineering projects:
   - Distributed Lease Engine
   - Real-Time Telemetry Processing
   - FastAPI Scalable Backend

4. **Resume** - Professional history with timeline, education, and skills
5. **Contact** - Contact form with social links

## Local Development

```bash
# Build the site
hugo --minify

# Serve locally
hugo server

# The site will be available at http://localhost:1313
```

## Deploying to GitHub Pages

1. Update `baseURL` in `hugo.toml` to your GitHub Pages URL:
   ```toml
   baseURL = 'https://yourusername.github.io/'
   ```

2. Build the site:
   ```bash
   hugo --minify
   ```

3. The built site is in the `public/` directory. Push this to your GitHub Pages repository.

## Customization

Edit `hugo.toml` to update:
- Author name and social links
- Site title and description
- GitHub/LikedIn usernames

## License

MIT
