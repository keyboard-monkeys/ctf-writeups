*Find the flag :)*

*no-right-click.hsc.tf*



<br>
The main point of the challenge is, that you can't use right click to open developer tools or view source. On Firefox, we can open developer tools with ctrl+shift+i. Then we can find, that there is a file referenced in head called useless-file.css. The contents of that file are:

```css
body {
    text-align: center;
    font-size: 5rem;
    font-family: 'Abril Fatface', cursive;
}

.small {
    margin-top: 50vh;
    font-size: 0.5rem;
}

/* cause i disabled it in index.js */
/* no right click = n.r.c. */
/* flag{keyboard_shortcuts_or_taskbar} */
```