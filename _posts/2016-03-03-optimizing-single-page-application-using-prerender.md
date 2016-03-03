---
layout: post
title: Optimizing Wildfly + Angularjs Single Page Applications For Faster Load Time
image: hyperdrive.jpg
date:  2016-02-03 12:00:00
---


<p class="intro"><span class="dropcap">T</span>he rise in capabilities of browsers and the prevalence of mobile apps javascript engine has fundamentally
altered the way we create websites.</p>
 
 
 
Not too long ago, we used to have most of our presentation logic reside in server side applications built on PHP or ruby
or java. Today, the cloud world is moving to a place where we write JSON endpoints in server side application and most of the presentation logic is handled by the client.
After Moore's law, we now have a new internet law <i>"Any application that can be written in JavaScript, will eventually be written in JavaScript"</i>. A look at the 
<a href="https://github.com/search?l=&o=desc&q=stars%3A%3E20000&ref=advsearch&s=stars&type=Repositories&utf8=%E2%9C%93"> popular repositories @ GitHub</a> will prove 
that javascript is the most popular language and single page applications rule the roost. Today javascript is no longer content with just the view layer of the web, it actually
transcends the boundary of Model and controller. Nodejs, angularjs and reactjs have stolen a march over conventional jquery+server side HTML websites.
 
When we created gozoomo, we wanted to give the best end user experience to our users. For [gozoomo.com](https://www.gozoomo.com), our production servers were
most a set of JSON endpoints and a place to serve static files. But using this stack also has a cost. We had a pure angular stack. Consequently, our servers 
render HTML with an empty body. After the browser loads the javascript libraries, it fetches the appropriate JSON. This meant that as discovered by 
[twitter](https://blog.twitter.com/2012/improving-performance-on-twittercom), many of our users did not engage with our website as they did not want to wait for
the entire javascript page to load before seeing a single character of what gozoomo is all about. Google page insights had a very sorry picture to tell us.
<figure>
	<img src="{{ '/assets/img/mobile.jpg' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Google pagespeed insights for mobile</figcaption>
</figure>
<figure>
	<img src="{{ '/assets/img/desktop.jpg' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Google pagespeed insights for desktop</figcaption>
</figure>

Building a great technology product is like surfing, if you have caught a popular wave, you will find some good soul solving the problems you are facing. The first
problem that came with Single page applications was that blank HTML could not be crawled by search engines. But this problem stood no chance against the combined might
of developer community. Soon, we had myriad of solutions to this. The solutions started with maintaining duplicate copies of server side and client side code. But code duplication was 
a nightmare and soon we had several libraries which would render the client side code by emulating a browser and serving the generated HTML. In this field, I think the clear 
winner is [prerender.io](https://prerender.io) which renders a cached version of the  pages. Prerender uses [phantomjs](http://phantomjs.org/) at their backend to generate HTML from javascript.
Airbnb came up with [rendr](http://nerds.airbnb.com/weve-launched-our-first-nodejs-app-to-product/). Rendr and prerender form the basis of my work.


## Optimization Step1: Eliminate render-blocking JavaScript in above-the-fold content

### Stage1: Serve HTML with partially filled body for SPA using widlfly

Rendr was pretty close to what we wanted. But we did not want to change our application from java and clojure to nodejs. So what we essentially wanted was a mechanism for me for to render a part of my HTML on
the server side and render the rest on the client side. My first instinct was to use the managed version of prerender which I was already using to serve search engines and other bots. I first tried using
the [java-prerender](https://github.com/greengerong/prerender-java) filter to route my requests to prerender.io. But I faced an issue, the filter was too specific for SEO purposes. There was no way for me to
serve all HTTP requests, but those originating from prerender, using prerender. Consequently, I used [AsyncHttpClient](https://github.com/AsyncHttpClient/async-http-client) with a timeout
to route requests to prerender depending on the user agent.I did not want my users to have to wait too long for prerender to generate the response and hence had a fallback mechanism.
{% highlight java %}

        if(userAgentString.contains("prerender")){
            return Response.ok().entity(new Viewable("/index.mustache", viewMap)).build();
        }else{
            SimpleAsyncHttpClient client = new SimpleAsyncHttpClient.Builder()
                    .setPooledConnectionIdleTimeout(100)
                    .setMaximumConnectionsTotal(50)
                    .setRequestTimeout(30 * 1000)
                    .setUrl(basePrerenderUrl+uriInfo.getAbsolutePath().toString()+(Boolean.TRUE.equals(mobileDevice)?"?mobileDevice=true":""))
                    .setHeader("Content-Type", "text/HTML").build();
            try{
                Future<com.ning.http.client.Response> f = client.get();
                return Response.ok().entity(f.get().getResponseBodyAsStream()).build();
            }catch (Throwable t){
                LOGGER.error("Exception fetching cached response",t);
                return Response.ok().entity(new Viewable("/index.mustache", viewMap)).build();
            }finally {
                client.close();
            }

        }
{% endhighlight %}

### Stage2: Fork Prerender and host locally to serve required HTML
This worked like a charm but there was a problem. Prerender used to strip the script tags from generated HTMLs. SEO haves have no need of the script tags but my customers sure needed them.
But the good folks at [prerender](https://github.com/prerender/prerender)  open sourced their library for all these use cases. They have a very nice modular structure to add and remove plugins.
I ran an instance of prerender on my machine after disabling the removeHtmlScript plugin. This unlocked my first achievement. Now, generating HTML using phantomjs is pretty time consuming, hence the next logical
step is using a form of caching. One good developer had already written a [plugin](https://github.com/jonathanbennett/prerender-redis-cache) for this. All I needed to do was change the 
[package.JSON and enable the plugin](https://github.com/youngmonk/prerender/commit/9f864a5ee59dc1051493af08837081f7fc15db5f#diff-b9cfc7f2cdf78a7f4b91a753d10865a2). By this time, I was using  
[foreman](https://github.com/strongloop/node-foreman) to run prerender. Hence, you also need to add `REDIS_URL="redis://seller.carcredible.com:6379"` to my .env file. This reduced the average time taken
to serve HTML to under 8 ms and also made my pre-cached website functional.

### Stage3: Optimizing initial HTML and handling things asynchronously
The next step was to make the javascript asynchronous. This was done by first removing the ng-app from the body tag and writing the following code as the last line of the body.
{% highlight html %}
<script>
    function jedi(){

        window.stateName='{{state_name}}';
        angular.bootstrap(document.getElementsByTagName("body")[0], ['{{ng_app}}']);
        window.prerenderReady = true;

    }
</script>
<link rel=stylesheet href="//static.gozoomo.com/web-assets/seller_static_assets/0.3.4-SNAPSHOT-3.{{ng_app}}.full.min.css"/>
<script async src="//static.gozoomo.com/web-assets/seller_static_assets/0.3.4-SNAPSHOT-3.app.full.min.js" onload="jedi('js')"></script>
{% endhighlight %}

### Stage4: Handle js exception and ignore parts of angular generated HTML

I kind of have and OCD to js exceptions. After optimization, angular typeahead plugin was giving me exceptions on page load. ["Angular bootstrap typeahead"](https://angular-ui.github.io/bootstrap/) expects to be connected to
a initialized array during compile phase of angular. Moreover, I wanted the HTML input elements to disabled till entire js is loaded.  Hence, I  needed a way for me to tell prerender to use a different variant of input tag.
Since I was only rendering a part of my HTML, I also used this variable to tell prerender to ignore some parts of HTML.This was done by using the following block of code.
{% highlight javascript %}
    $scope.preCached = true; // at the start of controller
    if($window.navigator.userAgent.indexOf('prerender')===-1){
            $scope.preCached = false;
        }// at the end of the controller
{% endhighlight %}
{% highlight html %}
<input type="text"
                   ng-model="model"
                   class="form-control"
                   placeholder="model"
                   autocomplete="off" ng-if="preCached" ng-disabled="preCached">
              <input type="text"
                     ng-model="model"
                     typeahead="mod for mod in (generic_cars || []) | filter:$viewValue | limitTo:4"
                     typeahead-editable="false"
                     class="form-control"
                     placeholder="model"
                     autocomplete="off" ng-if="!preCached">
{% endhighlight %}

## Optimization Step2: Eliminate render-blocking css in above-the-fold content

### Stage1: Find the CSS actually used in the HTML
This was arguably more tricky for me. As a rule, we only write SCSS and not CSS. As a rule, we do not like to duplicate code. As a rule, we do not write inlineCSS. But google recommendations
were asking us to write part of the CSS inline. Coming back to the surfing analogy, this was a problem faced by many a people and some great libraries had come up with takes an HTML string and CSS
file as input and emits the CSS part actually being used. I used [uncss](https://github.com/giakki/uncss) to find out the CSS being actually used by my cached HTML and then used 
[cleanCSS](https://github.com/jakubpawlowicz/clean-css) to minify the CSS. Since I had already dealt with prerender plugins,all I know had to do was to write a prerender plugin for this.The following code was used.
<script src="https://gist.github.com/himangshuj/108d78671673b821c45c.js"></script>

### Stage2: making CSS async
Async attribute that I used in javascript is not supported in the link tag. We had to make do with the media property of link to make CSS asynchronous. This also meant changes in my css prerender plugin and hence `(matches.indexOf("media")===-1)`.
{% highlight html %}
    <link rel=stylesheet href="//static.gozoomo.com/web-assets/seller_static_assets/0.3.4-SNAPSHOT-3.{{ng_app}}.full.min.css" media="bogus"/>
 <!-- Content goes here-->

    <link rel=stylesheet href="//static.gozoomo.com/web-assets/seller_static_assets/0.3.4-SNAPSHOT-3.{{ng_app}}.full.min.css"/>

{% endhighlight %}

## Optimization Step3: Enable Compression

Most modern browsers support gzip and it is better to make use of this  [compression](https://developers.google.com/speed/docs/insights/EnableCompression). Since we were using a standard undertow and wildfly. 
This was was fairly simple. I had to add the below entry in standalone.xml. This also took care of minifying HTML.
        {% highlight xml %}
         <server name="default-server">
                <http-listener name="default" redirect-socket="https" socket-binding="http"/>
                <host name="default-host" alias="localhost">
                    <location name="/" handler="welcome-content"/>
                    <filter-ref name="server-header"/>
                    <filter-ref name="x-powered-by-header"/>
                    <filter-ref name="gzipFilter" predicate="exists['%{o,Content-Type}'] and regex[pattern='(?:application/javascript|text/css|text/HTML|text/xml|application/json)(;.*)?', value=%{o,Content-Type}, full-match=true]"/>
                    <filter-ref name="Vary-header"/>
                </host>
            </server>
            <servlet-container name="default">
                <jsp-config/>
                <websockets/>
            </servlet-container>
            <handlers>
                <file name="welcome-content" path="${jboss.home.dir}/welcome-content"/>
            </handlers>
            <filters>
                <response-header name="server-header" header-value="WildFly/10" header-name="Server"/>
                <response-header name="x-powered-by-header" header-value="Undertow/1" header-name="X-Powered-By"/>
                <response-header name="Vary-header" header-value="Accept-Encoding" header-name="Vary"/>
                <gzip name="gzipFilter"/>
            </filters>
            {% endhighlight %}

## Optimization Step4: Leverage browser caching
[This](https://developers.google.com/speed/docs/insights/LeverageBrowserCaching) was also pretty straightforward and all I had to do was add a cookie. This ensures that for send request to the URL,
 the browser fetches from its cache instead of making a trip to the server.
           {% highlight java %}
            val calendar = Calendar.getInstance();
                   calendar.add(Calendar.DATE,5);
          
                   return Response.status(Response.Status.OK).entity(finalJson).expires(calendar.getTime()).build();
           {% endhighlight %}
## Optimization Step5: Viewport Issues

This was more of a bug. One carousel we were using had "overflow:visible" as CSS. This was causing part of HTML to go out of the viewport. This is not recommended. I just had to take care of the CSS.
           
           
# End Results
<figure>
	<img src="{{ '/assets/img/mobile1.jpg' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Google pagespeed insights for mobile</figcaption>
</figure>
<figure>
	<img src="{{ '/assets/img/desktop1.jpg' | prepend: site.baseurl }}" alt=""> 
	<figcaption>Google pagespeed insights for desktop</figcaption>
</figure>           

I could not hit 100% because of [segment](https://segment.com) libraries. But the results were damn good. To say thanks to all open-source contributors who have made this project possible. The code is open sourced 
@[prerender-fork](https://github.com/youngmonk/prerender)

