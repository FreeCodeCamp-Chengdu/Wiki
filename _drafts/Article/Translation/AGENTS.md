# Tech Article Translation Agent Guide

This directory contains English tech articles pending translation to Chinese. This document provides guidance for AI coding assistants.

## Translation Quality Standards

- **Accuracy**: Faithfully convey original meaning without omissions or distortions
- **Fluency**: Output should follow natural Chinese expression patterns
- **Consistency**: Maintain uniform terminology throughout the article
- **Completeness**: Preserve code examples, links, and document structure

## Chunked Translation Strategy (For 300+ Line Articles)

Long articles exceed single conversation context limits. Chunked processing ensures translation quality.

### Step 1: Analyze Article Structure

1. Read the full article, identify main sections (typically `##` headings)
2. Count total lines and lines per section
3. Divide article into translation units at section boundaries, each 200-300 lines

### Step 2: Build Glossary

Before translating, scan the full text to extract key terms:

```markdown
## Glossary

| English Term | Chinese Translation | Notes |
|-------------|---------------------|-------|
| Shadow DOM | 影子 DOM | Web Components term |
| Custom Element | 自定义元素 | |
| hydration | 水合 | React/SSR term |
```

Use this glossary as reference to ensure consistency across the article.

### Step 3: Translate by Chunks

For each translation unit:

1. **Read the section's original text**
2. **Translate with glossary reference**
3. **Edit the original file directly**, replacing English with Chinese
4. **Keep code blocks unchanged**
5. Proceed to next section

### Step 4: Integration Check

After translation:

1. Read through entire article for coherence
2. Verify terminology consistency
3. Update front matter
4. Move file to `_posts/Article/Translation/`

## Translation Unit Template

Each translation unit should be tracked as:

```
=== Translation Unit [N/Total] ===
Section: [Section Title]
Line Range: [Start-End]
Status: [Pending/In Progress/Completed]
===
```

## Content Cleanup Rules

Remove the following during translation:

- Navigation breadcrumbs (e.g., `- [Home][1] - [Docs][2]`)
- Author avatars and social links
- Page metadata prompts (e.g., "Stay organized with collections...")
- Duplicate titles (keep one)

## Content to Preserve

- All code blocks (keep as-is)
- `<!-- more -->` excerpt separator
- Footnote link references (`[1]: https://...`)
- Image references

## Translation Style Guide

### Terminology Handling

```
First occurrence: 服务端渲染（Server-Side Rendering，SSR）
Subsequent uses: 服务端渲染 or SSR
```

### Punctuation

- Use Chinese punctuation: ，。！？：；""''
- Use English punctuation for code-related content

### Paragraph Format

- Maintain original paragraph structure
- Split overly long sentences for readability when appropriate

## Example: Translation Workflow

Using `declarative-shadow-dom.md` (332 lines) as example:

```
1. Analyze structure:
   - Front Matter: lines 1-9
   - Introduction: lines 10-54 → Unit 1
   - Building a Declarative Shadow Root: lines 55-82 → Unit 2
   - Component hydration: lines 83-151 → Unit 3
   - Remaining sections... → Units 4-N

2. Build glossary:
   | Shadow DOM | 影子 DOM |
   | Declarative Shadow Root | 声明式影子根 |
   | Custom Element | 自定义元素 |
   | slot | 插槽 |

3. Translate unit by unit, editing file directly

4. Final review and move file
```

## Common Technical Terms Reference

| English | Recommended Chinese |
|---------|---------------------|
| Shadow DOM | 影子 DOM |
| Web Components | Web 组件 |
| Custom Elements | 自定义元素 |
| Server-Side Rendering | 服务端渲染 |
| hydration | 水合/激活 |
| streaming | 流式传输 |
| polyfill | 垫片/polyfill |
| slot | 插槽 |
| encapsulation | 封装 |
| DOM tree | DOM 树 |
| API | API |
| callback | 回调 |
| async/await | async/await |
| middleware | 中间件 |
| container | 容器 |
| orchestration | 编排 |
| deployment | 部署 |
| CI/CD | CI/CD |
