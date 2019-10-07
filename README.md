# My personal website

<https://www.burgestrand.se/>

GitHub pages, nothing fancy.

Jekyll theme originally from https://github.com/sergiokopplin/indigo. I've
adapted it for my needs slightly, and ported it to SCSS as to avoid any extra
tooling with GitHub pages.

## Guide

See `bin/develop` for writing new articles and developing the site.
You might want to `bundle update` every now and then.

## Development

```
Landing page (/)
├── Articles (/articles/)
│   ├── Page #<n> (/articles/page-<n>/)
│   └── Article (/articles/<title>)
└── About Me (/about-me/)
```