# Wiki pages (source)

Live wiki: **https://github.com/MrLouisWright/next-herald/wiki**

These files mirror `docs/` for the GitHub wiki. After editing `docs/`, regenerate them, then publish:

```bash
git clone https://github.com/MrLouisWright/next-herald.wiki.git
cp wiki/*.md next-herald.wiki/ && rm next-herald.wiki/README.md
cd next-herald.wiki && git add -A && git commit -m "Sync wiki" && git push
```

Sidebar links use GitHub wiki `[[Title|Slug]]` form.
