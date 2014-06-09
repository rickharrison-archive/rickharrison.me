---
layout: post
title: broccoli-asset-rev is now included in ember-cli
date: 2014-06-08 20:00:00 -0700
---

Recently, I published [broccoli-asset-rev](https://github.com/rickharrison/broccoli-asset-rev) on github, and it has been included in the most recent build of [ember-cli](http://iamstef.net/ember-cli/). When your ember application is built, all of your assets will be fingerprinted to prevent cache issues in the browser. Also, you can specify a URL to prepend to your assets, which can be used to add a CDN like cloudfront.

The end result turns

```javascript
<script src="assets/appname.js">
background: url('/images/foo.png');
```

into

```javascript
<script src="https://subdomain.cloudfront.net/assets/appname-342b0f87ea609e6d349c7925d86bd597.js">
background: url('https://subdomain.cloudfront.net/images/foo-735d6c098496507e26bb40ecc8c1394d.png');
```

It works by hashing the contents of your files and appending the hash to the end of the filename. Then, it parses all of your source files to update asset paths to the new location.

## Options

When using ember-cli, there are four options you can use to configure fingerprinting.

- `exclude` - Default: `[]` - An array of strings. If a filename contains any item in the exclude array, it will not be fingerprinted.

- `extensions` - Default: `['js', 'css', 'png', 'jpg', 'gif']` - The file types to add md5 checksums.

- `prepend` - Default: `''` - A string to prepend to all of the assets. Useful for CDN urls like `https://subdomain.cloudfront.net/`.

- `replaceExtensions` - Default: `['html', 'css', 'js']` - The file types to replace source code with new checksum file names.

## Sample Brocfile

ember-cli comes pre-packaged with sensible defaults, therefore you will often only need to implement 1 or 2 options. Here is an example for your `Brocfile.js`

```javascript
var app = new EmberApp({
  name: require('./package.json').name,

  minifyCSS: {
    enabled: true,
    options: {}
  },

  fingerprint: {
    exclude: ['fonts/169929'],
    prepend: 'https://sudomain.cloudfront.net/'
  },

  getEnvJSON: require('./config/environment')
});
```

If you have any questions, please contact me, and I will help in any way that I can.
