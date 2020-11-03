---
layout: post
title:      "How to spice up your Sinatra app with custom fonts"
date:       2020-11-03 01:17:38 +0000
permalink:  how_to_spice_up_your_sinatra_app_with_custom_fonts
---


You’ve created your database, the models + any associations, and the controllers for your project. Your HTML and CSS is OK. But your app works! After hours, maybe days, of debugging, you have a project you’re proud of. Well, sort of… after all, your app still looks like a form from 10 years ago (maybe worse).

Sound like you? A week ago, that was me. As I coded my Sinatra app, I found myself continuing to put off the HTML and CSS of my project until the very end (function first, right?). Consequently, I had an app that wasn’t very presentable.

One of the easiest ways you can differentiate your app from other is to change what people read – the font! Thus, one of the first things I researched was how to customize the font in my application. Here’s what you need to do.  

---

1\. First, the fun part, download a font. There are plenty of websites you can do this for free (if this is a commercial project, you’ll need to purchase a license, of course). Here are a few websites to get you started.

 - [Fontspace.com](http://fontspace.com)
 - [Dafont.com](http://dafont.com)
 - [Urbanfonts.com](http://urbanfonts.com)
 - [1001fonts.com](http://1001fonts.com)

2\. Next, your font file is probably something like .ttf, .otf, or .eot. It might even be a .woff or .woff2 file. But if you’ve downloaded your font from one of the above websites, chances are you probably have just one file type. You need at least a couple of different formats to ensure your font can be properly read across multiple browsers. To generate multiple types of font files with just one type of file, use a webfont generator. Two websites I used in my last project were,

 - [Font Squirrel]( https://www.fontsquirrel.com/tools/webfont-generator) – Use this first, which I think you can use for free an unlimited number of times. It will work for most of your font files.
 - [CloudConvert]( https://cloudconvert.com/). Font Squirrel might not work every time (a few times, the generator told me it wasn’t able to convert my files). For these cases, you can turn to an alternative free font generator. There are many out there, but CloudConvert worked fine for me. Note that there is a daily limit, so you may need to find another backup, which is no big deal (again, lots of free font generators out there).

Which type of files you need will depend on the level of browser support you want. Most browsers will support ‘woff’ and ‘woff2’, so these will probably be all you need. For more information on font file types and browser support, check out this great resource: https://css-tricks.com/snippets/css/using-font-face/.

3\.  Now, place your font files in the public directory of your application. By default, Sinatra will look here to serve your static files ([here’s how you can change the default location, if you want](http://sinatrarb.com/configuration.html)). I stored my fonts in a font directory one level below the public directory, like so (see the four .woff/.woff2 font files at the bottom):

```
|-- app
|-- config
|-- db
|-- public
|   |-- css
|   |   |
|   |   |_style.css
|   |
|   |-- fonts
|       |
|       |-- css
|       |   |
|       |   |_fonts.css
|       |
|       |-- fontA.woff
|       |-- fontA.woff2
|       |-- fontB.woff
|       |-- fontB.woff2
```

4\.  Almost there! Let’s get to some code. Using the example above, in the public/fonts/css/fonts.css file include the following @font-face rule, which will allow you to use your custom font files.

```
@font-face {
    font-family: 'some-memorable-font-family-name';
    src:  url('../fontA.woff2') format('woff2'),
            url('../fontA.woff') format('woff');
}
```

5\. Lastly, reference the font family name from the previous step in your stylesheet! Be sure to also include a default font, just in case a user accesses your app from a browser that doesn’t support your font file. For instance, in my public/css/style.css file, I might have something like,

```
.h1 {
     font-family: 'some-memorable-font-family-name', sans-serif;
}
```

That’s it! To take your font to the next level, add some colors. I found this resource to be helpful and easy to use: https://fossheim.io/writing/posts/css-text-gradient/ 

---

**Resources:**

- https://css-tricks.com/snippets/css/using-font-face/
- https://www.webdesignerdepot.com/2013/01/how-to-use-any-font-you-like-with-css3/
- https://fossheim.io/writing/posts/css-text-gradient/
