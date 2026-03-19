# Garden Content (Public)

Public content for my digital garden site.

## Structure

```
├── bookmarks/    # Saved links & resources
├── guides/       # Tutorials & how-tos
├── notes/        # General notes
├── projects/     # Project documentation
├── til/          # "Today I Learned" snippets
└── images/       # Media files
```

## How to Add Content

1. Create a markdown file in the appropriate folder
2. Add frontmatter:
   ```yaml
   ---
   title: "Your Title"
   description: "Brief description"
   ---
   ```
3. Write your content
4. Commit and push - site rebuilds automatically

## File Naming

- `til/` uses date prefix: `YYYY-MM-DD-title.md`
- Other folders: `kebab-case-title.md`
