# node-image-farmer

* Original connect-thumbs code by [Irakli Nadareishvili](https://github.com/inadarei), modified by [Allan Bogh](https://github.com/ajbogh)

Image thumbnailing middleware for Connect.js/Express.js that integrates with content-aware
cropping provided by [Smartcrop.js](https://github.com/jwagner/smartcrop.js/)

node-image-farmer implements the boilerplate code for creating thumbnails of large images in a standard,
Connect.js-complient way, allowing sensible defaults and high degree of customization.

## Installation

### Using your own API

    $ npm install node-image-farmer --save
    $ npm run install-ubuntu-deps
    $ # npm run install-mac-deps (untested)
    $ # npm run install-redhat-deps (untested)

### Using the provided API
    
    $ git clone git@github.com:ajbogh/node-image-farmer.git
    $ npm install
    $ npm run install-ubuntu-deps
    $ # npm run install-mac-deps (untested)
    $ # npm run install-redhat-deps (untested)
    
### Installing Dependencies (if "npm run install-XYZ-deps" doesn't work)   

node-image-farmer can use `GraphicsMagick` or `Imagemagick` for image manipulation
(see: [Configuration](##configuration)). 

Make sure your system has one of these packages properly installed, 
otherwise you are likely to get the following error: `Error: spawn identify ENOENT`. 

On OS-X you can easily install them with: 

```console
> brew install imagemagick
# and
> brew install graphicsmagick
# if you want webP support:
> brew install imagemagick --with-webp
```

Similarly, there are also APT and YUM repositories you can use for Ubuntu/Debian and 
RedHat/Centos/Fedora respectively.

If you are going to use smart (content-aware) cropping, you will also need to install Cairo. On OS-X you 
can install it with: 

```console
> xcode-select --install
> brew install pkgconfig
> brew install pixman
> brew install libjpeg
> brew install giflib 
> brew install cairo
```

The last step takes a while, and also: make sure everything links properly after each "brew install" and 
that you have the latest brew upgrade.

On other platforms, you can consult: [Cairo documentation](http://cairographics.org/download/).

## Serving local images

You can serve images from your own server by creating a symlink under app/images

```
$ cd app
$ ln -s images /path/to/your/images/folder
```

You can navigate your images folder similar to how you normally would with a URL:

```
http://localhost:3000/content/smart/small/my/subfolder/myImage.jpg
```

## API

*Port*: 3000 (default)
*Preset*: [full, small, medium, hero, irakli] (default, configurable)
*Width*: w or width (query string, optional)
*Height*: h or height (query string, optional)
*Quality*: q or quality (query string, optional, 1-100 default 95)

All images by default are served from port 3000 (configurable) and reside in the /content/smart subfolder
```
http://localhost:3000/content/smart
```

Presets are defined in the configuration, but default presets are 'irakli', 'small', 'medium', and 'hero'. 
If no preset matches or you would like the full image, you may use 'full' and optionally define width and height parameters.

```
http://localhost:3000/content/smart/small/myImage.jpg
http://localhost:3000/content/smart/medium/myImage.jpg
http://localhost:3000/content/smart/full/myImage.jpg
```

You may override preset crops with the w, width, h, or height parameters. You may also override the quality with the 'q' or 'quality' parameter.

```
http://localhost:3000/content/smart/small/myImage.jpg?w=150&h=100
http://localhost:3000/content/smart/medium/myImage.jpg?width=150&height=100
http://localhost:3000/content/smart/full/myImage.jpg?w=150&height=100&quality=50
http://localhost:3000/content/smart/full/myImage.jpg?q=2
```

## Running an Example

If you have all the prerequisites installed you can launch a demo with:

```
$ npm run app
```

And then open your browser at the [following URL](http://localhost:3000/content/smart/full/?base64=aHR0cDovL3d3dy5wdWJsaWNkb21haW5waWN0dXJlcy5uZXQvcGljdHVyZXMvMTAwMDAvdmVsa2EvMTA4MS0xMjQwMzI3MzE3cGMzcS5qcGc=):

```
http://localhost:3000/content/smart/full/?base64=aHR0cDovL3d3dy5wdWJsaWNkb21haW5waWN0dXJlcy5uZXQvcGljdHVyZXMvMTAwMDAvdmVsa2EvMTA4MS0xMjQwMzI3MzE3cGMzcS5qcGc=
```

You can see on the following diagram what simple (on the left), and smart (on the right)
 crops produce compared to the original (center)
 
 ![](https://raw.githubusercontent.com/inadarei/connect-thumbs/master/example/crops-smart.jpg)

Photo Credit: [Andrew Schmidt](http://www.publicdomainpictures.net/view-image.php?image=2514&picture=seagull&large=1) (Public Domain)

### Smart Cropping

200x400 remote image
```
http://localhost:3000/content/smart/full/?h=400&w=200&base64=aHR0cDovL3d3dy5wdWJsaWNkb21haW5waWN0dXJlcy5uZXQvcGljdHVyZXMvMTAwMDAvdmVsa2EvMTA4MS0xMjQwMzI3MzE3cGMzcS5qcGc=
```

200x400 local image
```
http://localhost:3000/content/smart/full/myImage.jpg?h=400&w=200
```
        
    
    
    
## Connect.js/Express.js Usage (Creating your own API)

    var thumbs = require('node-image-farmer');
    app.use(thumbs());
    
when configured with defaults, and if you have your node process running at yourdomain.com, a request such as:

    http://yourdomain.com/content/smart/medium/?base64=aHR0cDovL3VwbG9hZC53aWtpbWVkaWEub3JnL3dpa2lwZWRpYS9jb21tb25zLzYvNjYvRWluc3RlaW5fMTkyMV9ieV9GX1NjaG11dHplci5qcGc=
    
will display Einstein's photo from Wikipedia as a width: 300 (and proportionally resized height) thumbnail.

Another example uses the file system to resize a file within the app/images folder. In a production environment this folder would
be a symlink to wherever the images are located.

```
http://localhost:3000/content/smart/irakli/crops-smart.jpg
```

You may also override a preset by supplying a width, height, and quality in the query strings. Each of these are optional and
distinct properties. For instance, by specifying the width you can override the preset's width without affecting the height or quality properties.

```
http://localhost:3000/content/smart/default/crops-smart.jpg?height=300&width=200&quality=85
http://localhost:3000/content/smart/default/crops-smart.jpg?h=300&w=200&q=85
```


This is because:
 
1. `/content/smart/medium` in the begining of the URL instructs the middleware to use default resizing preset named "medium" 
 which corresponds to proportional resizing to width: 300px.
1. the long, somewhat cryptic code is base64-encoded version of the 
 [URL of Einstein's photo on Wikipedia](http://upload.wikimedia.org/wikipedia/commons/6/66/Einstein_1921_by_F_Schmutzer.jpg)
 and connect-middleware uses base64, by default, to encode the ID of the desired image.
 
You can provide an alternative `decodeFn` function, if you would rather use shorter IDs of your photos from your database, 
or UUIDs or whatever else makes sense to you (see below). Custom `decodeFn` functions must have following signature: 

    function(encodedURL, callback)
    
and must call callback, upon completion, with following syntax:

    callback(err, decodedURLValue);

## Configuration

```
    app.use(thumbs({
      "smartCrop" : false
    , "ttl" : 7200
    , "tmpCacheTTL" : 86400
    , "tmpDir" : "/tmp/mynodethumbnails"
    , "decodeFn" : someModule.loadImageUrlFromDbById
    , "allowedExtensions" : ['png', 'jpg']
    , "rootPath": "/thumbs"
    , "presets" : {
        small : {
          width: 120
          , quality:.5
        }
        , medium : {
          width: 300
          , quality:.7
        }
        , large : {
          width: 900
          , quality:.85
        }
      }
    }));
```

where:

 * smartCrop - enables experimental, content-aware cropping based on: [Smartcrop.js](https://github.com/jwagner/smartcrop.js/).
   This is `false` by default, until Smartcrop.js matures, but will become the default option as soon as
   there is a stable release of that project.
 * useIM - if you have trouble installing GraphicsMagick or prefer ImageMagick for any reason,
   setting this to 'true' will skip using GraphicsMagick and use ImageMagick instead. False by default.
 * ttl - is the client-side cache duration that will be returned in the HTTP headers for the resulting thumbnail.
 * tmpCacheTTL - time (in seconds) to cache thumbnails in temp folder. Defaults to 0 (cache disabled).
 * tmpDir - is the Node-writable temp folder where file operations will be performed. Defaults to: `/tmp/nodethumbnails`. 
   You may want to periodically clean-up that folder.
 * decodeFn - custom decoder function. Defaults to one that decodes base64-encoded full URLs.
 * allowedExtensions - file (path) extensions that node-image-farmer will try to thumbnail. Defaults to: jpg, jpeg, gif and png.
 * presets - json object describing various image presets. You can indicate width, height and quality 
   level for each. Quality adjusts image compression level and its value ranges from 0 to 100 (best).
    
    Currently width is required and it is the only required argument. Expect more flexibility here in 
    the following versions.

## Serving Behind a Web Server
    
*ATTENTION*: in typical web setups, static content such as images is often served by a web-server, never allowing 
requests to *.jpg, *.png etc. to reach Node process. If you want to use node-image-farmer, obviously you must allow
paths to thumbnailed images to pass through to Node. Please add appropriate exception to you web server configuration. 
For Nginx, your configuration may look something like the following:

```
  # Thumbnail processing
  location ^~ /content/smart {
    auth_basic off;

    proxy_pass         http://127.0.0.1:3000;
    proxy_set_header   Host                   $http_host;
    proxy_redirect off;
  }

  #  static content
  location ~* ^.+.(jpg|jpeg|gif|css|png|js|ico|xml)$ {
    # access_log        off;
    expires           15d;
  }
```

Alternatively, sometimes connect-static is used to serve static content. If you do that, please make sure that 
connect-static fires *after* node-image-farmer does.

## Performance and Scalability

Node.js is very fast, but Imagemagick and over-the-HTTP fetching of the original image most certainly are not. 
Neither may be your custom decodeFn function if it is doing a database lookup for every request. In any 
production setup it is highly recommended to put thubmnailing of images behind some sort of proxy/cache. 
Viable options include:

- Enabling the integrated disk-based cache provided by node-image-farmer. You can do this by passing custom `tmpCacheTTL`
configuration variable when initializing Thumbs. This variable is set in seconds and is 0 by default. Setting it 
to values greater than 0 enables caching.
- Put [Varnish](https://www.varnish-cache.org/) in front of the thumbnail URLs
- Use a robust CDN such as [Amazon's CloudFront](http://aws.amazon.com/cloudfront/)
- Pick your own poison.

## License

[MIT](LICENSE)
