# Agent instructions

This repo is a Jekyll blog published with GitHub Pages at https://evankuhn.github.io/.

## Markdown formatting

Wrap Markdown files (posts, pages, and docs like this one) at 100 characters per line.

- Break lines between words. Never split a word, URL, or link target across lines.
- A line may go past 100 characters only when it can't be broken, such as a long URL.
- Don't wrap YAML front matter, headings, table rows, or code blocks.
- Keep the blank lines between paragraphs and list items as they are.
- In a list item, indent continuation lines to line up with the item's text.

## File naming

Use lowercase words separated by hyphens for new files, including images: `jekyll-search.jpg`,
not `jekyll_search.jpg` or `JekyllSearch.jpg`. This matches the post URLs, and search engines
treat hyphens as word separators.

Don't rename files that are already published without asking. Renaming changes their URL, and
there's no way to redirect an image.

## Images in posts

- Put images in `images/`, named after the post with hyphens (e.g. `scaling-twitter.jpg`).
- Convert to JPEG, at most 1400px wide, with metadata stripped:
  `magick input.png -resize '1400x>' -quality 85 -strip images/<name>.jpg`
  (`1400x>` only shrinks, never enlarges.)
- Don't commit the original large PNG. Leave it outside the repo.
- A header image goes full-width at the top of the post, before the first paragraph or
  heading. Don't float wide or detailed images beside text; they become too small to read.
- Use Markdown with the image's real dimensions, and descriptive alt text wrapped at 100
  characters:
  `![Description of the image](/images/<name>.jpg){: width="1400" height="788"}`
- Also set `image: /images/<name>.jpg` in the front matter, so shared links use it as the
  preview.
- For a smaller figure with a caption, use the `centered-image` block:
  `<div class="centered-image"><img src="..." alt="..."><p>Caption</p></div>`

## Building and checking

To check a build, build into a temporary folder, not `_site/`:
`bundle exec jekyll build --destination "$(mktemp -d)"`

`_site/` is what a running `jekyll serve` shows, and `jekyll build` fills it with production
URLs.
