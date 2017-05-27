---
title: "Building my blog with Hugo"
type: "post"
description: "Documentation of how I learned to use and customize hugo and the tranquilpeaks theme"
date: 2017-05-27
publishdate: 2017-05-27
tags: [ "technology", "development", "java", "ansible", "python", "linux", "ubuntu" ]
categories: ["index", "technology", "development"]
keywords: [ "technology", "development", "index"]
clearReading: false
thumbnailImage: https://static.pexels.com/photos/147485/pexels-photo-147485.jpeg
thumbnailImagePosition: bottom
autoThumbnailImage: yes
metaAlignment: center
coverImage: https://static.pexels.com/photos/104916/pexels-photo-104916.jpeg
coverCaption: "A dreamy landscape"
coverMeta: out
coverSize: partial
comments: false
showTags: true
showPagination: true
showSocial: false
showDate: true
---

# Hugo
https://github.com/spf13/hugo/issues/1582
https://gohugo.io/commands/hugo_server/


# Hugo Theme

I decided to chose a pre-built theme : [tranquilpeak](https://github.com/kakawait/hugo-tranquilpeak-theme)
Useful documentation can be found here :
    * [User documentation](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/user.md)
    * [Developer documentation](https://github.com/kakawait/hugo-tranquilpeak-theme/blob/master/docs/developer.md)

I find that I am much more at ease to take an existing piece of code, understand how it works just enough to be able to modify it without breaking everything and then slowly making small changes to make it more to my liking. If I try to start something from scratch, I get lost trying to organise how I will structure my content / files and I end up not doing anything. Also, my knowledge of CSS is limited but its something I would really like to be good at.

I made a few (minor) changes to the theme.

## Sidebar photo
I ended up looking at royalty free stock photos [here](https://www.sitebuilderreport.com/stock-up), searched for something basic like "nature" and ended up with [this photo](https://www.goodfreephotos.com/albums/united-states/wisconsin/madison/wisconsin-madison-the-nature-boardwalk.jpg), credit to goodfreephotos.com.

## Sidebar css
By default, the icons and the text in the sidebar are a very pale color, almost white. I thought that it blended too much with the colors of the cover photo and I remember reading somewhere that any (darker) color of text with a white border around it really stands out no matter what the background is.

I searched a little and ended up finding this [alternate answer](https://stackoverflow.com/questions/2570972/css-font-border/8712442#8712442) with the [associated codepen](https://codepen.io/pixelass/pen/gbGZYL)

I added the function and the mixin at the end of `themes/tranquilpeak/src/scss/utils/mixins/_sidebar.scss` where themes is the folder at the root of the generated site and tranquilpeak is the folder where I cloned [the hugo-tranquilpeak project](https://github.com/kakawait/hugo-tranquilpeak-theme)

```
.sidebar-button-link {
    color:   map-get($sidebar, color);
    @include stroke(1, #FFFFFF);

    display: block;
    height:  100%;

    &:hover,
    &:active {
        text-decoration: none;
        color:           lighten(map-get($sidebar, color), 35);
    }
}
```

# Sidebar content height for xl screen size

I discovered a neat CSS trick for vertically centering any element in a page that I didn't know about.
Detailed explanation [here](https://stackoverflow.com/questions/25982135/why-does-left-50-transform-translatex-50-horizontally-center-an-element)

And here is the code for the hugo-tranquilpeak-theme in `src/scss/utils/mixins/_sidebar.scss`, the mixim in question is `@mixin sidebar-xlg`

```
.sidebar-container {
    position:          relative;
    top:               35%;
    transform:         translateY(-50%);
    -webkit-transform: translateY(-50%);
    -moz-transform:    translateY(-50%);
}
```

# Css text border
https://codepen.io/pixelass/pen/gbGZYL
https://stackoverflow.com/questions/2570972/css-font-border
