# Typst: hide footer page counter on some pages

Show page counter only from page 3 onwards:

```
#set page(
  [...]
  footer: context {
    if here().page() > 2 [
      #align(center, [
        #counter(page).display()
      ])
    ]
  }
)
```

Hide page counter on certain pages with a list:

```
    ...
    if here().page() not in (1, 2, 4, 7) [
    ...
```

Source: https://typst.app/docs/guides/page-setup-guide/#headers-and-footers
