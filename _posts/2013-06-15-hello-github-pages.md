---
layout: post
category : github
tags : [intro, jekyll, github]
---
{% include JB/setup %}

This Jekyll introduction will outline specifically  what Jekyll is and why you would want to use it.
Directly following the intro we'll learn exactly _how_ Jekyll does what it does.

## Initial Setup

After [installing jekyll](/index.html#start-now) you'll need to format your website directory in a way jekyll expects.
Jekyll-bootstrap conveniently provides the base directory format.

### The Jekyll Application Base Format

Jekyll expects your website directory to be laid out like so:

    .
    |-- _config.yml
    |-- _includes
    |-- _layouts
    |   |-- default.html
    |   |-- post.html
    |-- _posts
    |   |-- 2011-10-25-open-source-is-good.markdown
    |   |-- 2011-04-26-hello-world.markdown
    |-- _site
    |-- index.html
    |-- assets
        |-- css
            |-- style.css
        |-- javascripts


- **\_config.yml**  
	Stores configuration data.

- **\_includes**  
	This folder is for partial views.

- **\_layouts**   
	This folder is for the main templates your content will be inserted into.
	You can have different layouts for different pages or page sections.

- **\_posts**  
	This folder contains your dynamic content/posts.
	the naming format is required to be `@YEAR-MONTH-DATE-title.MARKUP@`.

- **\_site**  
	This is where the generated site will be placed once Jekyll is done transforming it. 

- **assets**  
	This folder is not part of the standard jekyll structure.
	The assets folder represents _any generic_ folder you happen to create in your root directory.
	Directories and files not properly formatted for jekyll will be left untouched for you to serve normally.

(read more: <https://github.com/mojombo/jekyll/wiki/Usage>)

## (Potentially) Useful Links
- [https://help.github.com/articles/setting-up-a-custom-domain-with-pages](https://help.github.com/articles/setting-up-a-custom-domain-with-pages)
- [http://jekyllbootstrap.com/](http://jekyllbootstrap.com/)
- [http://erjjones.github.io/blog/How-I-built-my-blog-in-one-day/](http://erjjones.github.io/blog/How-I-built-my-blog-in-one-day/)
- [http://jekyllrb.com/docs/variables/](http://jekyllrb.com/docs/variables/)
- [http://stackoverflow.com/questions/10685961/multiple-github-pages-and-custom-domains-via-dns/10766694#10766694](http://stackoverflow.com/questions/10685961/multiple-github-pages-and-custom-domains-via-dns/10766694#10766694)
