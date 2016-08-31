---
layout: post
title:  "Using color maps in SCSS"
date:   2016-08-17 09:15:04 -0400
permalink: /other/using-color-maps-in-scss/
category: design
---

SCSS is great. In 2016, I can't live without it. Gone are the days of plain old CSS, if you are not using a CSS preprocessor these days, you should try it right now, you won't regret it and you will not want to switch back!

You can use variables, functions, mixins and a bunch of other things. It's really great. One common idea is to create variables for the different colors your application will be using. But how do we make it better? Isn't there a solution to group the colors and simplify the variable names when we are calling them.

There is, it's called maps :)

# Why should you use Color maps

# How to use
{% highlight SCSS %}
$brandColors: (
    scaffold: (
        base:           #5c5c5c,
        error:          #B72B2C,
        newContact:     #50D749,
        logout:         #E32E30
    ),
    button: (
        base:           rgba(0, 133, 215, 0.6),
        shadow:         rgba(0, 133, 215, 1),
        hover:          #0085d7,
        hoverShadow:    darken(#0085d7, 8%)
    ),
    buttonDelete: (
        base:           rgba(227, 46, 48, 0.6),
        shadow:         rgba(227, 46, 48, 1),
        hover:          #e32e30,
        hoverShadow:    darken(#e32e30, 10%)
    ),
    pills: (
        base:           #008FED,
        family:         #FF387E,
        acquaintances:  #B5D158,
        following:      #DB5A3D
    )
);
{% endhighlight %}

But...

{% highlight SCSS %}
@function colors($color, $tone: "base") {
    @return map-get(map-get($brandColors, $color), $tone);
}
{% endhighlight %}

# Conclusion
