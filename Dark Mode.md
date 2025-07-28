# Dark Mode
On Linux Mint, some Applications (e.g. Firefox 141, Element) do not automatically switch to dark mode because the following dconf setting is not set when selecting a dark theme.

Execute manually:

```
dconf write /org/gnome/desktop/interface/color-scheme "'prefer-dark'"
```
