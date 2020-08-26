# images | lazy load

Created: Aug 25, 2020 4:13 AM
URL: https://dev.to/prototyp/best-way-to-lazy-load-images-for-maximum-performance-27o1

![images%20lazy%20load%207c66b80936b84a69a63cf05450659256/cqr1hz4w2vzas92rjvs7.png](images%20lazy%20load%207c66b80936b84a69a63cf05450659256/cqr1hz4w2vzas92rjvs7.png)

Image lazy loading is one of the more popular approaches of optimizing websites due to the relatively easy implementation and large performance gain. With **lazy loading** we load images asynchronously, meaning that we load images only when they appear in the browser's viewport.

Almost a year ago, native lazy loading for images and iframes was released for Chrome and other major browsers. The point of the feature is to give browsers control when to request an image or iframe resource, which makes dev jobs a bit easier. Up to that point, only option was to use various JavaScript plugins which monitored the viewport changes and loaded resources dynamically. Now browsers can do that natively!

At the time of writing this article, around [73% of currently used browsers](https://caniuse.com/#search=loading) support this feature which is not bad, but we don't want to make the website image content inaccessible and unusable to 27% of potential users.

So this puts us in an interesting situation:

- We want to use the benefits of native lazy loading for browsers that support it
- We want to use a JS plugin as fallback for lazy loading for browsers that don't support it
- We don't want to load the lazy loading JS plugin if the browser supports native lazy loading.
- Support both `img` and `source` elements is mandatory

## The "loading" attribute

We have three possible values that we can use for `loading` attribute.

- `auto` - default value. Same as not setting the attribute.
- `eager` - load the resource immediately.
- `lazy` - load the resource once it's in the viewport.

Although it depends on the use-case, generally we want to use `eager` value for resources above the fold and `lazy` value for resources below the fold.

## Modern approach

We need to write a script that will run after the HTML document is loaded. I've used Jekyll and added the script as an include that was appended to the end of the HTML `body` element. This is the most effective way of running JavaScript functions to avoid render blocking.

### Image markup

We want our JavaScript function to start the image loading process based on the native lazy loading feature support. To achieve that we'll add the path to our images to `data-src` instead of `src`. But we shouldn't leave `src` empty, so we'll use 1x1px transparent image placeholder. Our markup for `img` elements will look something like this

```
 <img 
    src="/path/to/placeholder/image.png"
    data-src="/path/to/full/image.jpg"
    alt="Image description"
    class="lazyload"
    loading="lazy"
/>

```

**Please note** that `class="lazyload"` is used by the lazyload fallback plugin. I've used [lazysizes](https://github.com/aFarkas/lazysizes) that uses this particular class name.

Additionally, we want to support `picture` element that contains multiple `source` element and fallback `img` element.

```

<picture>
    <source data-srcset="path/to/image.webp" type="image/webp" />
    <source data-srcset="path/to/image.jpg" />
    <img loading="lazy" 
        class="lazyload"
        src="path/to/placeholder/image.png"
        data-src="path/to/image.jpg"
        alt="Image description"
    />
</picture>

```

### Feature detection

We need to detect if user's browser supports native lazy loading. Luckily, we can do that using JavaScript directly.

```
   if ("loading" in HTMLImageElement.prototype) {
      /* Native lazy loading is supported */
   } else {
      /*  Native lazy loading is not supported */
   }

```

### Final JavaScript code

For **native lazy loading**, we only need to assign `data-src` value to `src` value for `img` and `source` elements and let the browser handle the rest.

For **unsupported browsers**, we only need to load the JavaScript plugin and, optionally, run it (if not done automatically). I've used [lazysizes](https://github.com/aFarkas/lazysizes) but any plugin will work, just make sure that the markup is correct (class names, data elements, etc.).

So the final JavaScript code will look something like this:

```
<script>
    if ("loading" in HTMLImageElement.prototype) {
        var images = document.querySelectorAll('img[loading="lazy"]');
        var sources = document.querySelectorAll("source[data-srcset]");
        sources.forEach(function (source) {
            source.srcset = source.dataset.srcset;
        });
        images.forEach(function (img) {
            img.src = img.dataset.src;
        });
    } else {
        var script = document.createElement("script");
        script.src = "/link/to/lazyload.js";
        document.body.appendChild(script);
    }
</script>

```

## Boosted performance & Lighthouse score

On my [personal website](https://codeadrian.github.io/) I've used a JavaScript plugin for image lazy loading for all browsers. After implementing this modern approach, I've eliminated one JavaScript file that is being loaded and parsed on website load which in turn boosted my Lighthouse score and overall performance!

## More image optimization techniques for maximum performance

Lazy loading is one of many ways to optimize image performance on the web. I've wrote this in-depth posts that covers other important techniques and aspects of image optimization for the web like web-specific image formats, using CDN, progressive images, etc.

These articles are fueled by coffee. So if you enjoy my work and found it useful, consider buying me a coffee! I would really appreciate it.

Thank you for taking the time to read this post. If you've found this useful, please give it a ❤️ or 🦄, share and comment.