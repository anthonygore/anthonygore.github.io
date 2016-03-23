---
layout: post
title:  "Retina Images With WordPress Using bfi_thumb"
date:   2015-04-08 00:00:00 +1100
tags: wordpress
permalink: /retina-images-wordpress-using-bfi_thumb/
----------------------------------------------------

If you want to serve retina images with WordPress plugin [WP Retina 2x](http://apps.meow.fr/wp-retina-2x/) but you’re using [bfi_thumb](https://github.com/bfintal/bfi_thumb), you’re going to have a slight problem.

bfi_thumb creates new file names that are not picked up by WP Retina 2x. For example, you may have an image file called your_image.jpg and WP Retina 2x may have prepared another called your_image@2x.jpg to serve on retina devices. But when you bring bfi_thumb to the party as well, it’ll create it’s own file e.g. your_image_resized.jpg. When your page is loading WP Retina 2x sees your_image_resized.jpg, it doesn’t have a retina-equivalent prepared.

I overcame this on my site by making a modification to bfi_thumb so it creates two resized thumbnails: your_image_resized.jpg and your_image_resized@2x.jpg. You can check out the code on [Github](https://github.com/anthonygore/bfi_thumb).
