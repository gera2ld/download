# downloadex

> Download and extract files

*See [download-cli](https://github.com/kevva/download-cli) for the command-line version.*

This is a drop-in replacement for the original [download](https://github.com/kevva/download), which many projects depend on to download binary files or third-party source code.

The problem for the original version is that it always tries to download the resources, without cache, and not able to use mirrors. This is terrible when used in slow / poor network environments, e.g. in China.

With this replacement, you can easily rewrite the URLs before downloading, by creating an `.downloadrc` file in your home directory. See [Rewriting](#Rewriting) for more details.

If you are using yarn, simply adding this to your `package.json` will work:

```js
{
  "resolutions": {
    // one of:
    "download": "https://registry.npmjs.org/downloadex/-/downloadex-8.0.3.tgz", // recommended for universal usage
    "download": "https://registry.npm.taobao.org/downloadex/download/downloadex-8.0.3.tgz",
    "download": "gera2ld/download#fb65cb5c", // commit hash can be cached
    "download": "gera2ld/download#v8.0.3", // tag will be fetched every time on install
  }
}
```

## Install

```
$ npm install downloadex
```


## Usage

```js
const fs = require('fs');
const download = require('downloadex');

(async () => {
	await download('http://unicorn.com/foo.jpg', 'dist');

	fs.writeFileSync('dist/foo.jpg', await download('http://unicorn.com/foo.jpg'));

	download('unicorn.com/foo.jpg').pipe(fs.createWriteStream('dist/foo.jpg'));

	await Promise.all([
		'unicorn.com/foo.jpg',
		'cats.com/dancing.gif'
	].map(url => download(url, 'dist')));
})();
```

### Proxies

To work with proxies, read the [`got documentation`](https://github.com/sindresorhus/got#proxies).

### Rewriting

This feature allows downloading from mirrors, by supporting URL rewriting.

Create a `.downloadrc` in you home directory or somewhere else that can be searched by [cosmiconfig](https://github.com/davidtheclark/cosmiconfig), with prefix replacing rules. For instance, rewriting the URLs to download image processing binary files:

```json
{
  "rewrite": {
    "https://raw.githubusercontent.com/imagemin/cwebp-bin/": "https://npm.taobao.org/mirrors/cwebp-bin/",
    "https://raw.githubusercontent.com/imagemin/mozjpeg-bin/": "https://npm.taobao.org/mirrors/mozjpeg-bin/",
    "https://raw.githubusercontent.com/imagemin/pngquant-bin/": "https://npm.taobao.org/mirrors/pngquant-bin/"
  }
}
```

By using a JavaScript config file `.downloadrc.js` instead of a JSON one, you can also provide `rewrite` as a function. For example, rewriting GitHub URLs to jsdelivr:

```js
module.exports = {
  rewrite(uri) {
    const matches = uri.match(/^https:\/\/raw\.githubusercontent\.com\/([^/]+\/[^/]+)\/([^/]+)\/(.*)$/);
    if (matches) {
      const [, repo, tag, path] = matches;
      return `https://cdn.jsdelivr.net/gh/${repo}@${tag}/${path}`;
    }
  },
};
```

## API

### download(url, destination?, options?)

Returns both a `Promise<Buffer>` and a [Duplex stream](https://nodejs.org/api/stream.html#stream_class_stream_duplex) with [additional events](https://github.com/sindresorhus/got#streams-1).

#### url

Type: `string`

URL to download.

#### destination

Type: `string`

Path to where your file will be written.

#### options

Type: `Object`

Same options as [`got`](https://github.com/sindresorhus/got#options) and [`decompress`](https://github.com/kevva/decompress#options) in addition to the ones below.

##### extract

Type: `boolean`<br>
Default: `false`

If set to `true`, try extracting the file using [`decompress`](https://github.com/kevva/decompress).

##### filename

Type: `string`

Name of the saved file.
