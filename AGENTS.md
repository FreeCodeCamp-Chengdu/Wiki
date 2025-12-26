# AGENTS.md - FreeCodeCamp Chengdu Community Wiki

This project is a documentation database, primarily used for translating English tech articles into Chinese.

## Project Structure

- `_drafts/Article/Translation/` - Articles pending translation
- `_posts/Article/Translation/` - Completed translations
- `_posts/` - Other published content

## Translation Workflow

1. Select an article from `_drafts/Article/Translation/`
2. Translate in the original file (do not create new files)
3. After completion, move the file to `_posts/Article/Translation/`

## Article Front Matter Format

After translation, update front matter to:

```yaml
---
title: Chinese title
date: Original article date
updated: Translation completion date
authors:
  - Translator's GitHub username
original: Original article URL
categories:
  - Article
  - Translation
toc: true
---
```

## Code Style

- Use Simplified Chinese
- For technical terms, use format on first occurrence: `中文术语（English Term）`
- Keep code blocks, commands, and variable names unchanged
- Translate link text, keep URLs unchanged
- Remove navigation links, author avatars, and irrelevant content from original
- Preserve `<!-- more -->` marker as excerpt separator

## Long Article Translation Strategy

For articles over 300 lines, refer to `_drafts/Article/Translation/AGENTS.md` for chunked translation strategy.
