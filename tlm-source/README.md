## Techies Like Me is all about:

* Responsive templates. Looking good on mobile, tablet, and desktop.
* Gracefully degrading in older browsers. Compatible with Internet Explorer 8+ and all modern browsers. 
* Minimal embellishments. Content first --- other widget nonsense never.
* Large feature images for posts and pages.
* Simple and clear permalink structure *(ie: domain.com/category/post-title)*


``` bash
tlm-source/
├── _includes
|    ├── author-bio.html  //bio stuff goes here
|    ├── chrome-frame.html  //displays on IE8 and less
|    ├── footer.html  //site footer
|    ├── head.html  //site head
|    ├── navigation.html //site top nav
|    └── scripts.html  //jQuery, plugins, GA, etc.
├── _layouts
|    ├── home.html  //homepage layout
|    ├── page.html  //page layout
|    ├── post-index.html  //post listing layout
|    └── post.html  //post layout
├── _posts
├── assets
|    ├── css  //preprocessed less styles. good idea to minify
|    ├── img  //images and graphics used in css and js
|    ├── js
|    |   ├── main.js  //jQuery plugins and settings
|    |   └── vendor  //all 3rd party scripts
|    └── less 
├── images  //images for posts and pages
├── about.md  //about page
├── articles.md  //lists all posts from latest to oldest
└── index.md  //homepage. lists 5 most recent posts
```


