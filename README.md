# pjax = pushState + ajax


            .--.
           /    \
          ## a  a
          (   '._)
           |'-- |
         _.\___/_   ___pjax___
       ."\> \Y/|<'.  '._.-'
      /  \ \_\/ /  '-' /
      | --'\_/|/ |   _/
      |___.-' |  |`'`
        |     |  |
        |    / './
       /__./` | |
          \   | |
           \  | |
           ;  | |
           /  | |
     jgs  |___\_.\_
          `-"--'---'

## Introduction

pjax is a jQuery plugin that uses ajax and pushState to deliver（传送、交付） a fast browsing experience with real permalinks（固定链接）, page titles, and a working back button.

pjax works by grabbing（抓取、爬虫类） html from your server via（通过） ajax and replacing the content
of a container on your page with the ajax'd html. It then updates the browser's
current URL using pushState without reloading your page's layout or any
resources (JS, CSS), giving the appearance(外貌、样子) of a fast, full page load. But really
it's just ajax and pushState.

For [browsers that don't support pushState][compat] pjax fully degrades(降低、降级).

## Overview

pjax is not fully automatic. You'll need to setup and designate（指定） a containing element on your page that will be replaced when you navigate your site.

Consider the following page.

``` html
<!DOCTYPE html>
<html>
<head>
  <!-- styles, scripts, etc -->
</head>
<body>
  <h1>My Site</h1>
  <div class="container" id="pjax-container">
    Go to <a href="/page/2">next page</a>.
  </div>
</body>
</html>
```

We want pjax to grab the URL `/page/2` then replace `#pjax-container` with
whatever it gets back. No styles or scripts will be reloaded and even the `<h1>`
can stay the same - we just want to change the `#pjax-container` element.

We do this by telling pjax to listen on `a` tags and use `#pjax-container` as the target container:

``` javascript
$(document).pjax('a', '#pjax-container')
```

Now when someone in a pjax-compatible（兼容的） browser clicks "next page" the content of `#pjax-container` will be replaced with the body of `/page/2`.

Magic! Almost. You still need to configure your server to look for pjax requests and send back pjax-specific content.

The pjax ajax request sends an `X-PJAX` header so in this example (and in most cases) we want to return just the content of the page without any layout for any requests with that header.

Here's what it might look like in Rails:

``` ruby
def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end
```

If you'd like a more automatic solution than pjax for Rails check out [Turbolinks][].

Also check out [RailsCasts #294: Playing with PJAX][railscasts].

## Installation

### bower

Via [Bower][]:

```
$ bower install jquery-pjax
```

Or, add `jquery-pjax` to your app's `bower.json`.

``` json
  "dependencies": {
    "jquery-pjax": "latest"
  }
```

### standalone

pjax can be downloaded directly into your app's public directory - just be sure you've loaded jQuery first.

```
curl -LO https://raw.github.com/defunkt/jquery-pjax/master/jquery.pjax.js
```

**WARNING** Do not hotlink the raw（原始、未加工的） script url. GitHub is not a CDN.

## Dependencies

Requires jQuery 1.8.x or higher.

## Compatibility（兼容性）

pjax only works with [browsers that support the `history.pushState`
API][compat]. When the API isn't supported pjax goes into fallback mode:
`$.fn.pjax` calls will be a no-op（停止操作指令） and `$.pjax` will hard load the given URL.

For debugging purposes（目的、为目的）, you can intentionally（故意地） disable pjax even if the browser supports `pushState`. Just call `$.pjax.disable()`. To see if pjax is actually supports `pushState`, check `$.support.pjax`.

## Usage

### `$.fn.pjax`

Let's talk more about the most basic way to get started:

``` javascript
$(document).pjax('a', '#pjax-container')
```

This will enable pjax on all links and designate the container as `#pjax-container`.

If you are migrating（迁移） an existing site you probably don't want to enable pjax everywhere just yet. Instead of using a global selector like `a` try annotating（注解、注释） pjaxable links with `data-pjax`, then use `'a[data-pjax]'` as your selector.

Or try this selector that matches any `<a data-pjax href=>` links inside a `<div data-pjax>` container.

``` javascript
$(document).pjax('[data-pjax] a, a[data-pjax]', '#pjax-container')
```

#### Arguments

The synopsis（摘要、总览） for the `$.fn.pjax` function is:

``` javascript
$(document).pjax(selector, [container], options)
```

1. `selector` is a string to be used for click [event delegation][$.fn.on].
2. `container` is a string selector that uniquely （唯一）identifies（可识别） the pjax container.
3. `options` is an object with keys described（描述的） below.

##### pjax options

key | default | description
----|---------|------------
`timeout` | 650 | ajax timeout in milliseconds after which a full refresh is forced
`push` | true | use [pushState][] to add a browser history entry upon navigation
`replace` | false | replace URL without adding browser history entry
`maxCacheLength` | 20 | maximum cache size for previous（先前的、以前的） container contents
`version` | | a string or function returning the current pjax version
`scrollTo` | 0 | vertical position to scroll to after navigation. To avoid（避免） changing scroll position, pass `false`.
`type` | `"GET"` | see [$.ajax][]
`dataType` | `"html"` | see [$.ajax][]
`container` | | CSS selector for the element where content should be replaced
`url` | link.href | a string or function that returns the URL for the ajax request
`target` | link | eventually the `relatedTarget` value for [pjax events](#events)
`fragment` | | CSS selector for the fragment（碎片） to extract（提取） from ajax response

You can change the defaults globally by writing to the `$.pjax.defaults` object:

``` javascript
$.pjax.defaults.timeout = 1200
```

### `$.pjax.click`

This is a lower level function used by `$.fn.pjax` itself. It allows you to get a little more control over the pjax event handling.

This example uses the current click context（上下文） to set an ancestor（祖先、被继承人） as the container:

``` javascript
if ($.support.pjax) {
  $(document).on('click', 'a[data-pjax]', function(event) {
    var container = $(this).closest('[data-pjax-container]')
    $.pjax.click(event, {container: container})
  })
}
```

**NOTE** Use the explicit（明确的） `$.support.pjax` guard（保卫、监视）. We aren't using `$.fn.pjax` so we should avoid binding this event handler unless（除非） the browser is actually going to use pjax.

### `$.pjax.submit`

Submits a form via pjax.（通过pjax提交表单）

``` javascript
$(document).on('submit', 'form[data-pjax]', function(event) {
  $.pjax.submit(event, '#pjax-container')
})
```

### `$.pjax.reload`

Initiates（启动、发起） a request for the current URL to the server using pjax mechanism（机制、机理） and replaces the container with the response. Does not add a browser history entry.

以当前的url通过ajax的方式向服务器发起一个请求，用服务器的响应来替换container中的内容，并且不添加浏览器历史纪录。

``` javascript
$.pjax.reload('#pjax-container', options)
```

### `$.pjax`

Manual（手动的） pjax invocation（体力）. Used mainly when you want to start a pjax request in a handler that didn't originate （创世、起源于）from a click. If you can get access to a click `event`, consider `$.pjax.click(event)` instead.

通常，当你想在一个处理函数中而不是从一个点击事件中发起一个pjax请求，你可以手动的调用pjax，注意，如果你有权访问一个click事件，请用`$.pjax.click(event)`代替它。

``` javascript
function applyFilters() {
  var url = urlForFilters()
  $.pjax({url: url, container: '#pjax-container'})
}
```

### Events

All pjax events except `pjax:click` & `pjax:clicked` are fired from the pjax
container, not the link that was clicked.

<table>
<tr>
  <th>event</th>
  <th>cancel</th>
  <th>arguments</th>
  <th>notes</th>
</tr>
<tr>
  <th colspan=4>event lifecycle upon following a pjaxed link</th>
</tr>
<tr>
  <td><code>pjax:click</code></td>
  <td>✔︎</td>
  <td><code>options</code></td>
  <td>fires from a link that got activated; cancel to prevent pjax</td>
</tr>
<tr>
  <td><code>pjax:beforeSend</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>can set XHR headers</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:send</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <td><code>pjax:clicked</code></td>
  <td></td>
  <td><code>options</code></td>
  <td>fires after pjax has started from a link that got clicked</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>before replacing HTML with content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:success</code></td>
  <td></td>
  <td><code>data, status, xhr, options</code></td>
  <td>after replacing HTML content loaded from the server</td>
</tr>
<tr>
  <td><code>pjax:timeout</code></td>
  <td>✔︎</td>
  <td><code>xhr, options</code></td>
  <td>fires after <code>options.timeout</code>; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:error</code></td>
  <td>✔︎</td>
  <td><code>xhr, textStatus, error, options</code></td>
  <td>on ajax error; will hard refresh unless canceled</td>
</tr>
<tr>
  <td><code>pjax:complete</code></td>
  <td></td>
  <td><code>xhr, textStatus, options</code></td>
  <td>always fires after ajax, regardless of result</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>xhr, options</code></td>
  <td></td>
</tr>
<tr>
  <th colspan=4>event lifecycle on browser Back/Forward navigation</th>
</tr>
<tr>
  <td><code>pjax:popstate</code></td>
  <td></td>
  <td></td>
  <td>event <code>direction</code> property: &quot;back&quot;/&quot;forward&quot;</td>
</tr>
<tr>
  <td><code>pjax:start</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>before replacing content</td>
</tr>
<tr>
  <td><code>pjax:beforeReplace</code></td>
  <td></td>
  <td><code>contents, options</code></td>
  <td>right before replacing HTML with content from cache</td>
</tr>
<tr>
  <td><code>pjax:end</code></td>
  <td></td>
  <td><code>null, options</code></td>
  <td>after replacing content</td>
</tr>
</table>

`pjax:send` & `pjax:complete` are a good pair of events to use if you are implementing a
loading indicator. They'll only be triggered if an actual XHR request is made,
not if the content is loaded from cache:

``` javascript
$(document).on('pjax:send', function() {
  $('#loading').show()
})
$(document).on('pjax:complete', function() {
  $('#loading').hide()
})
```

An example of canceling a `pjax:timeout` event would be to disable the fallback
timeout behavior if a spinner is being shown:

``` javascript
$(document).on('pjax:timeout', function(event) {
  // Prevent default timeout redirection behavior
  event.preventDefault()
})
```

### Server side

Server configuration will vary between languages and frameworks. The following example shows how you might configure Rails.

``` ruby
def index
  if request.headers['X-PJAX']
    render :layout => false
  end
end
```

An `X-PJAX` request header is set to differentiate a pjax request from normal XHR requests. In this case, if the request is pjax, we skip the layout html and just render the inner contents of the container.

[Check if there is a pjax plugin][plugins] for your favorite server framework.

#### Response types that force a reload

By default, pjax will force a full reload of the page if it receives one of the
following responses from the server:

* Page content that includes `<html>` when `fragment` selector wasn't explicitly
  configured. Pjax presumes that the server's response hasn't been properly
  configured for pjax. If `fragment` pjax option is given, pjax will simply
  extract the content to insert into the DOM based on that selector.

* Page content that is blank. Pjax assumes that the server is unable to deliver
  proper pjax contents.

* HTTP response code that is 4xx or 5xx, indicating some server error.

#### Affecting the browser URL

If the server needs to affect the URL which will appear in the browser URL after
pjax navigation (like HTTP redirects work for normal requests), it can set the
`X-PJAX-URL` header:

``` ruby
def index
  request.headers['X-PJAX-URL'] = "http://example.com/hello"
end
```

#### Layout Reloading

Layouts can be forced to do a hard reload when assets or html changes.

First set the initial layout version in your header with a custom meta tag.

当静态资源和html改变的时候，layout可以被强制重新加载，首先，你需要在header中用一个自定义的meta标签设置layout的初始版本。

``` html
<meta http-equiv="x-pjax-version" content="v123">
```

Then from the server side, set the `X-PJAX-Version` header to the same.

然后，在服务端，把响应头设置为相同的`X-PJAX-Version`

``` ruby
if request.headers['X-PJAX']
  response.headers['X-PJAX-Version'] = "v123"
end
```

Deploying a deploy, bumping（碰撞、使撞击） the version constant（常量） to force clients to do a full reload the next request getting the new layout and assets.

[compat]: http://caniuse.com/#search=pushstate
[$.fn.on]: http://api.jquery.com/on/
[$.ajax]: http://api.jquery.com/jQuery.ajax/
[pushState]: https://developer.mozilla.org/en-US/docs/Web/Guide/API/DOM/Manipulating_the_browser_history#Adding_and_modifying_history_entries
[plugins]: https://gist.github.com/4283721
[turbolinks]: https://github.com/rails/turbolinks
[railscasts]: http://railscasts.com/episodes/294-playing-with-pjax
[bower]: https://github.com/twitter/bower
