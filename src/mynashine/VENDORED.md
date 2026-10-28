# Vendored MynaShine

This directory is a copy of the portable core from
[MynaShine](https://github.com/jhicksdev/mynashine), the syntax highlighter used for
fenced code blocks in MynaPress.

- Upstream commit: `5846ca4` (`Target released myna-lang 0.5.0`)
- Upstream version: 0.1.0
- License: MIT (same as MynaPress)

Files are copied unmodified. Do not edit them here; fix issues upstream and re-sync.

## What is copied

| File | Purpose |
| :--- | :--- |
| `util.myna` | Portable string, character, and HTML escape helpers |
| `token.myna` | Token model, span merging, lossless rendering |
| `myna_lexer.myna` | Context-aware Myna scanner |
| `lang_py.myna` | Python scanner |
| `lang_js.myna` | JavaScript scanner |
| `lang_config.myna` | TOML and YAML scanners |
| `lang_shell.myna` | Bash/shell scanner |
| `lang_markup.myna` | HTML, XML, Markdown, and template scanners |
| `highlight.myna` | Public `highlight_code` / `highlight_block` / `highlight_html_document` / `default_css` API |
| `theme.myna` | Theme CSS generation |

The CLI (`main`, `cli`, `fsutil`, `site_proc`, `demo`) is intentionally not vendored.
MynaPress calls `highlight_block` directly from `src/markdown.myna`, so it does not need
the post-processor or filesystem helpers.

## Syncing

From a checkout of both projects:

```bash
cd /path/to/mynashine
git checkout <upstream-commit>
for f in util token myna_lexer lang_py lang_js lang_config lang_shell lang_markup highlight theme; do
  cp "src/$f.myna" /path/to/mynapress/src/mynashine/"$f.myna"
done
```

Update the commit and version above, then refresh the stylesheet block between
`/* mynashine:start */` and `/* mynashine:end */` in `assets/style.css` from
`default_css("mynapress")`. Run the MynaPress test suite and build the documentation site
afterwards.
