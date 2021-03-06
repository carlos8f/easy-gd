_The documentation on 0.0.7 version (which is installed when you do ``npm install easy-gd``) could be found at [npmjs.org/package/easy-gd](https://npmjs.org/package/easy-gd)._

_The master branch has been heavily rewritten, the documentation will follow._

# easy-gd [![Build Status](https://travis-ci.org/furagu/easy-gd.png?branch=master)](https://travis-ci.org/furagu/easy-gd)

Easy-gd is a node.js wrapper of the [GD image manipulation library](http://libgd.bitbucket.org) designed with the following goals in mind:

1. Provide an organic node.js interface to the libgd library having C-style interface itself.
2. Automate the routine image manipulation tasks like resizing, cropping, watermarking, etc.

## User Guide
This documentation begins with some examples of what could be done with easy-gd, then provides complete details on the features.

* [Quickstart](#quickstart)
    * [Making Thumbnails](#making-thumbnails)
    * [Adding Watermarks](#adding-watermarks)
    * [Opening and Saving Buffers](#opening-and-saving-buffers)
    * [Reading and Writing Streams](#reading-and-writing-streams)
* [Advanced Topics](#advanced-topics)
    * [The Library Design](#the-library-design)
    * [Opening Images](#opening-images)

## Quickstart

### Making Thumbnails

Let's start with everyday basics. The most common thing to do with a graphics library on the web is to make a thumbnail of a user-uploaded image. Let's do it step by step with easy-gd.

```js
var gd = require('easy-gd')          // load the library

gd.open('uploaded-image.jpg')        // open the image
  .resize({width: 100, height: 100}) // resize it to feat into 100x100-pixel square
  .save('image-thumbnail.jpg')       // save the resized image
```

That's it! The image was shrunk to feat into 100x100-pixel square and was saved into image-thumbnail.jpg file.

The production-ready solution, though, should be made in a slightly different manner to take the advantage of asynchronous node.js nature. Read the section on [asynchronous processing](#TODO).

### Adding Watermarks

Adding a watermark is something that often goes hand by hand with image resizing. A watermark is a semi-transparent image to be put on top of the original image to represent a website or company attribution.

The common flow is something like this:

* User uploads an image.
* The image gets scaled down to some reasonable dimentions.
* A watermark gets applied to the image.
* The image gets saved and served.

This is how to do the resize-watermark-save part with easy-gd:

```js
var gd = require('easy-gd')

gd.open('uploaded.jpg')
  .resize({width: 1000, height: 1000})
  .watermark('logo.png')
  .save('large.jpg')
```

The default watermark position is the center of the image. To specify the position pass an additional __pos__ argument to the .watermark() call.

```js
image = image.watermark('logo.png', {x: 0, y: 0}) // the left top corner
image = image.watermark('logo.png', {x: 1, y: 1}) // the right bottom corner
image = image.watermark('logo.png', {x: 0.5, y: 1}) // at the bottom, centered horizontally
```

Read the [advansed section](#TODO) to make a watermarking process asynchronous.

### Opening and Saving Buffers

Sometimes the image comes in the form of buffer containing the image data. Easy-gd handles buffers just the same way it handles files: gd.open checks the type of the source passed in and does the right thing.

```js
var gd = require('easy-gd')

var imageBuffer = getTheImageBuffer() // fs.readFileSync(filename) for example

gd.open(imageBuffer)
  .resize({width: 1000, height: 1000})
  .save(filename)
```

Sometimes it is necessary to get the resulting image as a buffer containing the image data. With easy-gd it is pretty easy, too: just call the image.save() without parameters and you'll get the buffer.

```js
var gd = require('easy-gd')

var resizedBuffer = gd.open('image.jpg')
  .resize({width: 500, height: 500})
  .save() // this is the actual method returning a buffer
```

Also, a buffer could be specified as the watermark source in the [image.watermark()](#adding-watermarks) method.

### Reading and Writing Streams

The streams are the right way to deal with the data flows in node.js. Easy-gd supports streams in three ways, which are reading, writing and pipelining.

To read a stream just pass it to the gd.open() method. Due to asynchronous stream nature, you should specify a callback to handle the image when it is ready.

```js
var gd = require('easy-gd')

var stream = getInputStream() // fs.createReadStream(filename) or process.stdin

gd.open(stream, function (error, image) {
    if (error) {
        console.log(error)
        return
    }
    image.resize({width: 1000, height: 1000}).save('resized.jpg')
})
```

Stream writing, as you maybe already guessed, is performed by the image.save() method. Again, you should provide a callback because streams are asynchronous.

```js
var gd = require('easy-gd')

var stream = getOutputStream() // fs.createWriteStream(filename) or process.stdout

gd.open('image.jpg')
  .resize({width: 1000, height: 1000})
  .save(stream, function (error) { // note the callback
    console.log(error || 'success')
  })
```

Sometimes both the input and the output are represented as a stream. For that case node.js has a concept of [transform stream](http://nodejs.org/api/stream.html#stream_class_stream_transform), which is implemented in easy-gd.

Just omit the .open() and .save() calls to make a transform stream and then pipeline it to the source and the destination.

```js
var gd = require('easy-gd')

var input  = getInputStream()  // fs.createReadStream(filename) or process.stdin
var output = getOutputStream() // fs.createWriteStream(filename) or process.stdout

var transform = gd.resize({width: 1000, height: 1000}).watermark('logo.png')
input.pipe(transform).pipe(output)
```

## Advanced Topics

### The Library Design

The library is built on top of the [node-gd](https://www.npmjs.org/package/node-gd), which itself is a node.js wrapper of the [GD image manipulation library](http://libgd.bitbucket.org).

Easy-gd is designed to be a wrapper of the underlying node-gd, extending the functionality of node-gd with new features. This wrapping is made in three aspects:

1. All of the node-gd methods present in easy-gd. You can just replace ```require('node-gd')``` with ```require('easy-gd')``` and everything works as before.
2. gd.open() returns ordinary node-gd images (gd.Image descendants), so you can use your existing image processing methods if any.
3. All the image objects returned by node-gd have extended methods like resize(), save() and other, so one can generate images with some existing code and save them with image.save() or resize with image.resize().

The philosophy of easy-gd is to make the routine image processing tasks easy and consistent.

For instance, gd.open() will open an image in any [format it knows](#supported-formats) from any type of the source it knows, be it a stream, a file or a buffer. It will return the image sincronously if one does not provide a callback, which is handy for proof-of-concept prototyping. It will also return the image asynchronously being passed a callback, so, when one is ready to write an asynchronous production code, the method stays exactly the same, only the callback agreement is added.

All of the easy-gd methods are made this way, hiding the boring details of detecting formats, calculating the image size, reading buffers and streams and more.

### Opening Images

```js
gd.open( source [, options ] [, callback] )
```

Open an image and return the image object.

__source__<br>
Type: String, Stream, or Buffer<br>
An image filename, a readable stream or a buffer containing the image data.

__options__<br>
Type: Object<br>
A set of key/value pairs that configure gd.open features.

* __autoOrient__ (default: true) Automatically orient the image using the Exif orientation tag (see a note on the [Exif handling](#TODO-exif-handling)).

__callback__<br>
Type: Function( gd.Error error, gd.Image image )<br>
A function to be called when the image is ready. The error argument is set to gd.Error descendant to indicate a failure or null to indicate a success (see the [Error Handling](#TODO-error-handling)).

#### Details

<a name="supported-formats"/>
The format of the image is automatically detected by checking the image contents. Only JPEG, PNG and GIF are supported.

If there's no callback given, gd.open() will synchronously return an image or will raise a [gd.Error](#TODO-error-handling) exception to indicate a failure.

The returned image is a gd.Image descendant, being an in-memory representation of the image used by GD library (one could spy the methods of the Image object in the gd.h header file of the C-library, typically located in /usr/local/include/gd.h).

With easy-gd the gd.Image gains some extra properties and methods.

The properties are:

* __format__ (String). Stores the initial image format label, could be 'jpeg', 'png' or 'gif'.
* __exif__ (Object). A set of key/value pairs representing the Exif tags found in the image (see the details on [Exif handling](#TODO-exif-handling)).

The methods are:

* __autoOrient()__                                    Automatically orient the image using the image.exif.Orientation tag (as gd.open() does by default).
* __crop( [ options ] [, callback] )__                Crop the image (see [Resizing Images](#TODO-resizing-images-crop)).
* __resize( [ options ] [, callback] )__              Resize the image proportionally (see [Resizing Images](#TODO-resizing-images)).
* __save( [ target ] [, options] [, callback] )__     Save the image to a file, a buffer or a stream (see [Writing Images](#TODO-writing-images)).
* __watermark( source [, position ] [, callback] )__  Add a watermark to the image (see [Adding Watermarks](#TODO-adding-watermarks)).

See the [Handy Image Methods](#TODO-handy-image-methods) for some built-in GD image functionality.

#### Examples

_TO BE CONTINUED_

## License

Copyright (C) 2013-2014 Egor Balyshev<br>
Copyright (C) 2013 Alexander Schepanovski

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
