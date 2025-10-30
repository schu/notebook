# Pandoc: convert Markdown to PDF (with Typst)

[Pandoc](https://pandoc.org/) supports Typst as [pdf-engine](https://pandoc.org/MANUAL.html#option--pdf-engine)
as an alternative to the default LaTeX engine. For example:

```
pandoc -M mainfont:'Helvetica Neue' --pdf-engine typst IN.md -o OUT.pdf
```

`-M mainfont:'...'` can be used to set the font. Currently, the `mainfont`
parameter is needed [to work around a `font fallback list must not be empty` error](https://github.com/jgm/pandoc/issues/11238)
with Typst v0.14.
