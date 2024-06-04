## Integrate React app into Jekyll

### Build react app

Firstly build your react app by running this command in your app working directory.

```
npm run build
```

This will place static site index.html, index.css and index.js into /dist directory.

### Move css and js files into jekyll

After building react app you can move index.js to /assets/js/ and index.css to assets/css/ in your jekyll app project.

### Create page for your application

You might want to place your react app under _project/ folder. To do this create react_app_name.md in under the _project directory/.

Then fill the markdown file with header:

```
---
layout: none
title: app-title
description: app-description
img: assets/some-image.jpg (thumbnail)
importance: 1
category: work
related_publications: true
---

<-- Put index.html content under the header. You might have to update <script> and <link> with new paths. -->
```

### If site uses any assets you should update path from index.js into domain/assets/

## Troubleshoot

If deployment fails it might be due to uglifier. Add below clausule to specific secion in _config.yml:
```
jekyll-minifier:
  exclude: ["robots.txt", "path_to_your_js_file"]
  uglifier_args:
    harmony: true
```