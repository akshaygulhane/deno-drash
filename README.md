# Drash

Drash is a modular web app framework for [Deno](https://deno.land) that respects RESTful design principles.

Drash helps you build web apps that handle requests to grab resources. Requests can request any representation of a resource (e.g., application/json, text/html, application/xml, etc.) as long as the resource allows it.

## Features
* Uses HTTP resources (not controllers) to process HTTP requests
* Content negotation (comes standard with handling `JSON`, `HTML`, and `XML`)
* Path params (e.g., `/uri/with/some/:id`)
* Semantic HTTP method routing (e.g., define `GET()` in your resource class to handle `GET` requests)

## Quickstart

### Step 1 of 6: Install Deno

Installation instructions can be found here: [https://deno.land/](https://deno.land/)

### Step 2 of 6: Make Your App Directory And Download Drash

```
$ mkdir app
$ cd app
$ git clone https://github.com/crookse/deno-drash.git drash
```

### Step 3 of 6: Create An HTTP Resource File

**File: `app/home_resource.ts`**

```typescript
import Drash from "./drash/mod.ts";

/** Define an HTTP resource that handles HTTP requests to the / URI */
export default class HomeResource extends Drash.Http.Resource {
  static paths = [
    '/',
    '/:name',
  ];

  /**
   * Handle GET requests.
   */
  public GET() {
    this.response.body = `Hello, ${this.request.path_params.name ? this.request.path_params.name : 'world'}!`;

    return this.response;
  }

  /**
   * Handle POSTS requests.
   */
  public POST() {
    this.response.body = 'POST request received!';
    if (this.request.path_params.name) {
      this.response.body = `Hello, ${this.request.path_params.name}! Your POST request has been received!`;
    }

    return this.response;
  }
}

```

### Step 4 of 6: Create Your App File

**File: `app/app.ts`**

```typescript
import Drash from "./drash/mod.ts";
import HomeResource from "./home_resource.ts";

let server = new Drash.Http.Server({
  response_output: 'text/html',
  resources: [
    HomeResource
  ]
});

server.run();
```

### Step 5 of 6: Run Your App

```
$ deno app.ts --allow-net
```

### Step 6 of 6: Make An HTTP Request

* Go to: `localhost:8000/`
* Go to: `localhost:8000/hello`
* Go to: `localhost:8000/hello/`
* Go to: `localhost:8000/hello/:name`
* Go to: `localhost:8000/hello/:name/`

## Things To Know

Drash servers use `Drash.Http.Response` to generate responses and send them to clients. It can generate responses of the following content types:

* `application/json`
* `application/xml`
* `text/html`
* `text/xml`

If you want your Drash server to handle more content types, then you will need to override `Drash.Http.Response`. See steps below to override `Drash.Http.Response`:

*Note: The following steps assume you're using the example code above.*

### Step 1 of 2: Make Your `Response` Class.

*Note: This class only needs to override the `send()` method.*

**File: `app/response.ts`**

```typescript
import Drash from "./drash/mod.ts";

/** Response handles sending a response to the client making the HTTP request. */
export default class Response extends Drash.Http.Response {
  /**
   * @overrides `Drash.Http.Response.send()`
   * 
   * Send a response to the client.
   */
  public send(): void {
    let body;

    switch (this.headers.get('Content-Type')) {
      // Handle HTML
      case 'text/html':
        body = this.body;
        break;

      // Handle JSON
      case 'application/json':
        body = JSON.stringify({body: this.body});
        break;

      // Handle PDF
      case 'application/pdf':
        this.headers.set('Content-Type', 'text/html');
        body = `<html><body style="height: 100%; width: 100%; overflow: hidden; margin: 0px; background-color: rgb(82, 86, 89);"><embed width="100%" height="100%" name="plugin" id="plugin" src="https://www.adobe.com/content/dam/acom/en/security/pdfs/AdobeIdentityServices.pdf" type="application/pdf" internalinstanceid="19"></body></html>`;
        break;

      // Handle XML
      case 'application/xml':
      case 'text/xml':
        body = `<body>${this.body}</body>`;
        break;

      // Handle plain text
      case 'text/plain':
        body = this.body;
        break;

      // Default to this
      default:
        body = this.body;
        break;
    }

    this.request.respond({
      status: this.status_code,
      headers: this.headers,
      body: new TextEncoder().encode(body),
    });
  }
}

```

### Step 2 of 2: Modify Your App File

**File: `app/app.ts`**

```typescript
 import Drash from "./drash/mod.ts";
+
+import Response from "./response.ts";
+Drash.Http.Response = Response;
+
+
 import HomeResource from "./home_resource.ts";

 let server = new Drash.Http.Server({
   response_output: 'text/html',
   resources: [
     HomeResource
   ]
 });

 server.run();
```

---

TODO:
* [ ]  Request URL parser
