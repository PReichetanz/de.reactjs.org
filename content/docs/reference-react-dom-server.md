---
id: react-dom-server
title: ReactDOMServer
layout: docs
category: Reference
permalink: docs/react-dom-server.html
---

Mit dem `ReactDOMServer`-Objekt können Komponenten als statisches Markup gerendert werden. Normalerweise wird es auf einem Node-Server verwendet:

```js
<<<<<<< HEAD
// ES-Module
import ReactDOMServer from 'react-dom/server';
=======
// ES modules
import * as ReactDOMServer from 'react-dom/server';
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b
// CommonJS
var ReactDOMServer = require('react-dom/server');
```

## Übersicht {#overview}

<<<<<<< HEAD
Die folgenden Methoden können sowohl in Server- als auch in Browserumgebungen verwendet werden:
=======
These methods are only available in the **environments with [Node.js Streams](https://nodejs.dev/learn/nodejs-streams):**

- [`renderToPipeableStream()`](#rendertopipeablestream)
- [`renderToNodeStream()`](#rendertonodestream) (Deprecated)
- [`renderToStaticNodeStream()`](#rendertostaticnodestream)

These methods are only available in the **environments with [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API)** (this includes browsers, Deno, and some modern edge runtimes):

- [`renderToReadableStream()`](#rendertoreadablestream)

The following methods can be used in the environments that don't support streams:
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

- [`renderToString()`](#rendertostring)
- [`renderToStaticMarkup()`](#rendertostaticmarkup)

<<<<<<< HEAD
Diese zusätzlichen Methoden benötigen ein Package (`stream`), das **nur auf dem Server verfügbar** ist und im Browser nicht funktionieren wird:

- [`renderToNodeStream()`](#rendertonodestream)
- [`renderToStaticNodeStream()`](#rendertostaticnodestream)

* * *

## Referenz {#reference}
=======
## Reference {#reference}
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

### `renderToPipeableStream()` {#rendertopipeablestream}

```javascript
ReactDOMServer.renderToPipeableStream(element, options)
```

<<<<<<< HEAD
Rendert ein React-Element initial als HTML. React gibt einen HTML-String zurück. Du kannst diese Methode verwenden, um HTML auf dem Server zu generieren und beim ersten Request das Markup zurückzusenden, damit die Seite schneller lädt und Suchmaschinen zu SEO-Zwecken deine Seiten crawlen können.

Wenn du auf einen Knoten, der bereits dieses server-gerenderte Markup hat, [`ReactDOM.hydrate()`](/docs/react-dom.html#hydrate) aufrufst, wird React das Markup behalten und nur Eventhandler hinzufügen. Das ermöglicht ein sehr schnelles erstes Laden der Seite.
=======
Render a React element to its initial HTML. Returns a stream with a `pipe(res)` method to pipe the output and `abort()` to abort the request. Fully supports Suspense and streaming of HTML with "delayed" content blocks "popping in" via inline `<script>` tags later. [Read more](https://github.com/reactwg/react-18/discussions/37)

If you call [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on a node that already has this server-rendered markup, React will preserve it and only attach event handlers, allowing you to have a very performant first-load experience.

```javascript
let didError = false;
const stream = renderToPipeableStream(
  <App />,
  {
    onShellReady() {
      // The content above all Suspense boundaries is ready.
      // If something errored before we started streaming, we set the error code appropriately.
      res.statusCode = didError ? 500 : 200;
      res.setHeader('Content-type', 'text/html');
      stream.pipe(res);
    },
    onShellError(error) {
      // Something errored before we could complete the shell so we emit an alternative shell.
      res.statusCode = 500;
      res.send(
        '<!doctype html><p>Loading...</p><script src="clientrender.js"></script>'
      );
    },
    onAllReady() {
      // If you don't want streaming, use this instead of onShellReady.
      // This will fire after the entire page content is ready.
      // You can use this for crawlers or static generation.

      // res.statusCode = didError ? 500 : 200;
      // res.setHeader('Content-type', 'text/html');
      // stream.pipe(res);
    },
    onError(err) {
      didError = true;
      console.error(err);
    },
  }
);
```

See the [full list of options](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-dom/src/server/ReactDOMFizzServerNode.js#L36-L46).

> Note:
>
> This is a Node.js-specific API. Environments with [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API), like Deno and modern edge runtimes, should use [`renderToReadableStream`](#rendertoreadablestream) instead.
>
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

* * *

### `renderToReadableStream()` {#rendertoreadablestream}

```javascript
ReactDOMServer.renderToReadableStream(element, options);
```

<<<<<<< HEAD
Ähnlich wie [`renderToString`](#rendertostring), außer dass es keine extra React-internen DOM-Attribute wie z. B. `data-reactroot` generiert. Das ist nützlich, wenn du React dazu verwenden willst, eine einfache statische Seite zu generieren, denn ohne diese extra Attribute können einige Bytes gespart werden.

Falls du planst, auf dem Client React zu benutzten, um das Markup interaktiv zu gestalten, solltest du diese Methode nicht benutzen. Verwende stattdessen [`renderToString`](#rendertostring) auf dem Server und [`ReactDOM.hydrate()`](/docs/react-dom.html#hydrate) auf dem Client.
=======
Streams a React element to its initial HTML. Returns a Promise that resolves to a [Readable Stream](https://developer.mozilla.org/en-US/docs/Web/API/ReadableStream). Fully supports Suspense and streaming of HTML. [Read more](https://github.com/reactwg/react-18/discussions/127)

If you call [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on a node that already has this server-rendered markup, React will preserve it and only attach event handlers, allowing you to have a very performant first-load experience.

```javascript
let controller = new AbortController();
let didError = false;
try {
  let stream = await renderToReadableStream(
    <html>
      <body>Success</body>
    </html>,
    {
      signal: controller.signal,
      onError(error) {
        didError = true;
        console.error(error);
      }
    }
  );
  
  // This is to wait for all Suspense boundaries to be ready. You can uncomment
  // this line if you want to buffer the entire HTML instead of streaming it.
  // You can use this for crawlers or static generation:

  // await stream.allReady;

  return new Response(stream, {
    status: didError ? 500 : 200,
    headers: {'Content-Type': 'text/html'},
  });
} catch (error) {
  return new Response(
    '<!doctype html><p>Loading...</p><script src="clientrender.js"></script>',
    {
      status: 500,
      headers: {'Content-Type': 'text/html'},
    }
  );
}
```

See the [full list of options](https://github.com/facebook/react/blob/14c2be8dac2d5482fda8a0906a31d239df8551fc/packages/react-dom/src/server/ReactDOMFizzServerBrowser.js#L27-L35).

> Note:
>
> This API depends on [Web Streams](https://developer.mozilla.org/en-US/docs/Web/API/Streams_API). For Node.js, use [`renderToPipeableStream`](#rendertopipeablestream) instead.
>
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

* * *

### `renderToNodeStream()`  (Deprecated) {#rendertonodestream}

```javascript
ReactDOMServer.renderToNodeStream(element)
```

<<<<<<< HEAD
Rendert ein React-Element initial als HTML. Gibt einen [Readable stream](https://nodejs.org/api/stream.html#stream_readable_streams) zurück, dessen Ausgabe ein HTML-String ist. 
Die HTML-Ausgabe dieses Streams ist exakt die gleiche wie die von [`ReactDOMServer.renderToString`](#rendertostring). Du kannst diese Methode verwenden, um HTML auf dem Server zu generieren und beim ersten Request das Markup zurückzusenden, damit die Seite schneller lädt und Suchmaschinen zu SEO-Zwecken deine Seiten crawlen können.

Wenn du auf einen Knoten, der bereits dieses server-gerenderte Markup hat, [`ReactDOM.hydrate()`](/docs/react-dom.html#hydrate) aufrufst, wird React das Markup behalten und nur Eventhandler hinzufügen. Das ermöglicht ein sehr schnelles erstes Laden der Seite.
=======
Render a React element to its initial HTML. Returns a [Node.js Readable stream](https://nodejs.org/api/stream.html#stream_readable_streams) that outputs an HTML string. The HTML output by this stream is exactly equal to what [`ReactDOMServer.renderToString`](#rendertostring) would return. You can use this method to generate HTML on the server and send the markup down on the initial request for faster page loads and to allow search engines to crawl your pages for SEO purposes.

If you call [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on a node that already has this server-rendered markup, React will preserve it and only attach event handlers, allowing you to have a very performant first-load experience.
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

> Hinweis:
>
> Nur für den Server. Diese API ist im Browser nicht verfügbar.
>
> Die Stream-Ausgabe dieser Methode gibt einen utf-8 kodierten Bytestream zurück. Falls du einen anders kodierten Stream benötigst, schau dir z. B. das Projekt [iconv-lite](https://www.npmjs.com/package/iconv-lite) an, das Transformationsstreams bereitstellt, um Text umzukodieren.

* * *

### `renderToStaticNodeStream()` {#rendertostaticnodestream}

```javascript
ReactDOMServer.renderToStaticNodeStream(element)
```

Ähnlich wie [`renderToNodeStream`](#rendertonodestream), außer dass es keine extra React-internen DOM-Attribute wie z. B. `data-reactroot` generiert. Das ist nützlich, wenn du React dazu verwenden willst, eine einfache statische Seite zu generieren, denn ohne diese extra Attribute können einige Bytes gespart werden.

Die HTML-Ausgabe dieses Streams ist exakt die gleiche wie die von  [`ReactDOMServer.renderToStaticMarkup`](#rendertostaticmarkup).

<<<<<<< HEAD
Falls du planst, auf dem Client React zu benutzten, um das Markup interaktiv zu gestalten, solltest du diese Methode nicht benutzen. Verwende stattdessen [`renderToNodeStream`](#rendertonodestream) auf dem Server und [`ReactDOM.hydrate()`](/docs/react-dom.html#hydrate) auf dem Client.
=======
If you plan to use React on the client to make the markup interactive, do not use this method. Instead, use [`renderToNodeStream`](#rendertonodestream) on the server and [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on the client.
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b

> Hinweis:
>
> Nur für den Server. Diese API ist im Browser nicht verfügbar.
>
<<<<<<< HEAD
> Die Stream-Ausgabe dieser Methode gibt einen utf-8 kodierten Bytestream zurück. Falls du einen anders kodierten Stream benötigst, schau dir z. B. das Projekt [iconv-lite](https://www.npmjs.com/package/iconv-lite) an, das Transformationsstreams bereitstellt, um Text umzukodieren.
=======
> The stream returned from this method will return a byte stream encoded in utf-8. If you need a stream in another encoding, take a look at a project like [iconv-lite](https://www.npmjs.com/package/iconv-lite), which provides transform streams for transcoding text.

* * *

### `renderToString()` {#rendertostring}

```javascript
ReactDOMServer.renderToString(element)
```

Render a React element to its initial HTML. React will return an HTML string. You can use this method to generate HTML on the server and send the markup down on the initial request for faster page loads and to allow search engines to crawl your pages for SEO purposes.

If you call [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on a node that already has this server-rendered markup, React will preserve it and only attach event handlers, allowing you to have a very performant first-load experience.

> Note
>
> This API has limited Suspense support and does not support streaming.
>
> On the server, it is recommended to use either [`renderToPipeableStream`](#rendertopipeablestream) (for Node.js) or [`renderToReadableStream`](#rendertoreadablestream) (for Web Streams) instead.

* * *

### `renderToStaticMarkup()` {#rendertostaticmarkup}

```javascript
ReactDOMServer.renderToStaticMarkup(element)
```

Similar to [`renderToString`](#rendertostring), except this doesn't create extra DOM attributes that React uses internally, such as `data-reactroot`. This is useful if you want to use React as a simple static page generator, as stripping away the extra attributes can save some bytes.

If you plan to use React on the client to make the markup interactive, do not use this method. Instead, use [`renderToString`](#rendertostring) on the server and [`ReactDOM.hydrateRoot()`](/docs/react-dom-client.html#hydrateroot) on the client.
>>>>>>> 84ad3308338e2bb819f4f24fa8e9dfeeffaa970b
