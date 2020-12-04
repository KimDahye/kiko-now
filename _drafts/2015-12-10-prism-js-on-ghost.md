---
layout: post
title: "Code block highlighting with prism.js on Ghost"
tags: []
comments: true
---


## how to?
-  다운로드 받기 - [여기](http://prismjs.com/download.html)에서 원하는 언어와 기능들을 선택한 후 js파일과 css파일을 다운받는다.
-  ghost 서버에 가서 `content/themes/theme-name/assets/js/prism.js`, `content/themes/theme-name/assets/css/prism.css` 위치에 파일을 둔다.
- 이제 이 파일들을 layout에 넣는다.
  * `content/themes/theme-name/default.hbs` 파일을 연다.
  * add `<link>` tag in the head for the style and add `<script>` tag before the closing body tag.

```language-hbs
{{! Styles'n'Scripts }}
...
<link rel="stylesheet" type="text/css" href="{{asset "css/prism.css"}}" />  
...

```

```language-hbs
...
    <script type="text/javascript" src="{{asset "js/prism.js"}}"></script>
</body> 
```

- node.js 재시작. ex) `pm2 restart index.js` 

## refernece
* http://blog.weareevermore.com/add-syntax-highlighting-to-ghost-with-prism-js/
