---
layout: post
title:  "Comet/WebSocket? Introducing the Atmosphere framework"
date:   2012-03-28 19:00:00 +0100
categories: ajax api atmosphere comet framework java jee jersey jquery push realtime webpush websocket
excerpt_separator: <!--more-->
---
Pushing messages to connected clients has always been a need on the web, growing with the apparition of new Rich Internet Applications, like **realtime feeds** (Gmail, news, market quotes), **social feeds** (The Twitter, Facebook and consort) and many other providing **realtime collaboration**, **monitoring** and **control** like new Internet of Things applications ([FlightRadar24](http://www.flightradar24.com), Arduino [#1](http://kevinrohling.wordpress.com/2011/09/14/world-domination-using-arduinos-and-websockets/), [#2](http://laclefyoshi.blogspot.com/2010/12/controlling-arduino-with-ipod-touch.html)).

[Comet](http://en.wikipedia.org/wiki/Comet_(programming)) technics (ie. polling, long-polling, streaming) and more recently, the [WebSocket](http://en.wikipedia.org/wiki/WebSocket) protocol, have made possible various webpush applications.

Today, when you want to enable realtime push on your own Java-based webapp, you have several solutions:

- Use an **externalized system** that will handle that for you (like [Pusher](http://pusher.com/)), avoiding you the providing of any additional server
- Use a **commercial solution** (like [Kaazing](http://kaazing.com/) or [Diffusion](http://www.pushtechnology.com/)), generally implementing a Message Broker underneath, with both embedded and standalone server
- Use a **container-specific api** (like [Jetty Continuation](http://docs.codehaus.org/display/JETTY/Continuations), Tomcat’s [CometProcessor](http://tomcat.apache.org/tomcat-6.0-doc/aio.html), [Grizzly CometHandler](http://grizzly.java.net/nonav/docs/docbkx2.2/html/grizzly-docs.html) or [Netty WebSocket](http://docs.jboss.org/netty/3.2/xref/org/jboss/netty/example/http/websocket/package-summary.html)), but your application will be strongly coupled with it
- Or use one of the few Open Source frameworks available (like Atmosphere, cometD or JWebSocket)

In this article, I will focus on **Atmosphere**. After a brief presentation of the framework, I will demonstrate how easy it is to make push-capable applications, whatever the container you use and the nature of your clients.

Before we start, you can take a look at the following video, that demonstrates what I achieved thanks to the Atmosphere framework (more details below, of course):

<iframe width="560" height="315" src="https://www.youtube.com/embed/1Abv88t5igc" frameborder="0" allowfullscreen></iframe>

<!--more-->

# What is Atmosphere ?

Atmosphere is a **Java OpenSource framework** led by Jean-Francois Arcand. Started around 2009, it evolved significantly and powers today websites like [smartmoney.com](http://www.smartmoney.com/). The project is very active (daily commits, lively mailing list, 300 Github followers). Currently in version 0.9, Atmosphere is close to the major 1.0 release planned for May.

One of the strength of the framework is that it is **container agnostic**: you can deploy an Atmosphere application in Jetty, Tomcat, JBoss, Grizzly,… Indeed, Atmosphere will seamlessly use specific compatibility modules and enable WebSocket depending on the detected container. The user does not need to learn container-specific Comet or WebSocket implementations anymore, only the way Atmosphere runs.

Atmosphere can run in two modes: **embedded** or **standalone**. The first one is the classic approach and the one you generally choose when you start from scratch. The second one, called Nettosphere, relies on the combination of the Netty container and the Atmosphere framework, allowing you to extend your existing Java webapp with realtime push capabilities in a non-intrusive way. Moreover, Nettosphere fits well when you need to do integration test on your realtime resources as you can instantiate it programmatically on demand, similarly to the Jersey-test-framework.

Usually, when you want to achieve standard **comet** (long-polling, streaming) communication, you don’t have to worry about the underneath container: they are compatible with Atmosphere. About **WebSocket** nonetheless, the compatibility will depend on the version of the WebSocket specification implemented by the container running Atmosphere. To this day, Jetty, Grizzly, Netty and even Tomcat are some WebSocket-capable containers.

A variety of **modules and plugins** are bundled with the framework, letting the user benefit from Atmosphere in various way: you are able to write Atmosphere applications in Java, Scala, JRuby and Groovy, and use other frameworks such as Jersey, GWT, Spring or JSF.

Another interesting thing: Atmosphere was built to scale thanks to **clustering plugins**. These plugins actually wrap a clustering layer implementation among JMS, Redis, XMPP or Hazelcast to name a few…  So if one server is not enough, you have tools to scale-out as needed.

Among all Atmosphere’s components, we find the **jQuery plugin**: this Javascript API connects your web pages to your server and offers methods to exchange realtime data. It can detect client’s capabilities and switch between protocols as necessary via a fallback mechanism. Therefore, the same user experience can be offered from IE6 to Chrome18.

From now on, we will focus on the Jersey’s Atmosphere extension, very convenient as it largely reduces lines to code thanks to JAX-RS implementation (annotations, json mapping, etc…).

# Grab your keyboard, launch your Eclipse!

To demonstrate the use of Atmosphere, I chose to build an application that draws realtime events on a map: user can generate events by clicking on the map, and server can generate random events. A generated event (by either client or server) is delivered to each connected clients.

Thus, this application – called MapPush (I was not able to find anything more explicit!) – is composed of:

- The **client**, a simple HTML page that uses Javascript/jQuery to process logic
- The **server**, a JEE webapp backed by Atmosphere and Jersey frameworks

*Note: the application is hosted on Github at the address [http://github.com/ncolomer/MapPush](http://github.com/ncolomer/MapPush). All the snippets below are extracted from this project. Feel free to browse the source code or clone the project!*

# Project bootstrap

In Eclipse, create a Maven project with a “webapp” archetype. As Atmosphere is available on Sonatype repositories ([snapshots](https://oss.sonatype.org/content/repositories/snapshots/org/atmosphere/), [releases](https://oss.sonatype.org/content/repositories/releases/org/atmosphere/)), you can simply add the following to your `pom.xml`:

{% highlight xml %}
<!-- Sonatype repositories -->
<repositories>
	<repository>
		<id>Sonatype snapshots</id>
		<url>https://oss.sonatype.org/content/repositories/snapshots</url>
	</repository>
	<repository>
		<id>Sonatype releases</id>
		<url>https://oss.sonatype.org/content/repositories/releases</url>
	</repository>
</repositories>

<!-- Dependencies -->
<dependencies>
	<!-- Atmosphere -->
	<dependency>
		<!-- Atmosphere's Comet Portable Runtime (CPR) -->
		<groupId>org.atmosphere</groupId>
		<artifactId>atmosphere-runtime</artifactId>
		<version>0.9.7</version>
	</dependency>
	<dependency>
		<!-- Atmosphere's Jersey module -->
		<!-- Transitivity will pull all necessary dependencies -->
		<!-- ie. Jersey 1.10 core, server, etc... -->
		<groupId>org.atmosphere</groupId>
		<artifactId>atmosphere-jersey</artifactId>
		<version>0.9.7</version>
	</dependency>
	<dependency>
		<!-- Atmosphere's jQuery plugin -->
		<groupId>org.atmosphere</groupId>
		<artifactId>atmosphere-jquery</artifactId>
		<version>0.9.7</version>
		<type>war</type>
	</dependency>
	<!-- Jersey's JSON mapper -->
	<dependency>
		<groupId>com.sun.jersey</groupId>
		<artifactId>jersey-json</artifactId>
		<version>1.10</version>
	</dependency>
</dependencies>
{% endhighlight %}

To start the framework we need a servlet, and not any… an AtmosphereServlet! In case the Atmosphere’s Jersey module is detected at load time, the framework will wrap around the Jersey Servlet (ContainerServlet) to load our resources and extend it with Atmosphere capabilities (IoC, annotations, etc…). Therefore, Jersey’s init-params are still available.

Open the `web.xml` file of your project and paste the following snippet:

{% highlight xml %}
<!-- Atmosphere -->
<servlet>
    <description>AtmosphereServlet</description>
    <servlet-name>AtmosphereServlet</servlet-name>
    <servlet-class>org.atmosphere.cpr.AtmosphereServlet</servlet-class>
    <init-param>
        <!-- Jersey base package of your resources -->
        <param-name>com.sun.jersey.config.property.packages</param-name>
        <param-value>org.mappush.resource</param-value>
    </init-param>
    <init-param>
        <!-- Enable Jersey's JSON mapping feature -->
        <param-name>com.sun.jersey.api.json.POJOMappingFeature</param-name>
        <param-value>true</param-value>
    </init-param>
    <load-on-startup>0</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>AtmosphereServlet</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
{% endhighlight %}

We are now ready to code both client and server application.

# The server

If you have already used the official JAX-RS implementation, Jersey, you won’t be disapointed. For others, you’ll find out that programming Atmosphere push APIs is really… dead simple!

Among the essential pieces of Atmosphere, one is called [`Broadcaster`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/Broadcaster.html). This is the object that manages connected clients and delivers broadcasted messages to them. A connected client is seen as an [`AtmosphereResource`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/AtmosphereResource.html) by the framework.

In the push communication lifecycle, we can define three steps:

- the client sends a request, that is suspended (in case of comet) or upgraded (in case of websocket) by the server.
- then, server and client can exchange data : server generally broadcasts/pushes data to clients, and in case of a duplex protocol like WebSocket, client can send data back to it.
- finally, either one closes the connection.

Atmosphere offers two convenient ways to handle this lifecycle: you can choose between **annotation** or **programmatic API** (or even a combination of both), the last one allowing more customization from my point of view.

- To suspend a request, use the [`@Suspend`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/annotation/Suspend.html) annotation, or simply return a [`SuspendResponse`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/jersey/SuspendResponse.html)
- To broadcast data, use the  [`@Broadcast`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/annotation/Broadcast.html) annotation and simply return a [`Broadcastable`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/jersey/Broadcastable.html). For a more specific use of broadcast mechanism, you are able to manually deal with the [`Broadcaster`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/Broadcaster.html) and its `broadcast(…)` methods to push messages to all, a subset or a particular [`AtmosphereResource`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/AtmosphereResource.html).

Here is an exemple of Atmosphere resource using the main concepts we have described above:

{% highlight java %}
@Path("/")
@Singleton
public class EventResource {

	private final Logger logger = LoggerFactory.getLogger(EventResource.class);

	private EventListener listener;
	private EventGenerator generator;

	private @Context BroadcasterFactory bf;

	/**
	 * Programmatic way to get a Broadcaster instance
	 * @return the MapPush Broadcaster
	 */
	private Broadcaster getBroadcaster() {
		return bf.lookup(DefaultBroadcaster.class, "MapPush", true);
    }

	/**
	 * The @PostConstruct annotation makes this method executed by the
	 * container after this resource is instanciated. It is one way
	 * to initialize the Broadcaster (e.g. by adding some Filters)
	 */
	@PostConstruct
	public void init() {
		logger.info("Initializing EventResource");
		BroadcasterConfig config = getBroadcaster().getBroadcasterConfig();
		config.addFilter(new BoundsFilter());
		listener = new EventListener();
		generator = new EventGenerator(getBroadcaster(), 100);
	}

	/**
	 * When the client connects to this URI, the response is suspended or
	 * upgraded if both client and server arc WebSocket capable. A Broadcaster
	 * is affected to deliver future messages and manage the
	 * communication lifecycle.
	 * @param res the AtmosphereResource (injected by the container)
	 * @param bounds the bounds (extracted from header and deserialized)
	 * @return a SuspendResponse
	 */
	@GET
	@Produces(MediaType.APPLICATION_JSON)
	public SuspendResponse<String> connect(@Context AtmosphereResource res,
			@HeaderParam("X-Map-Bounds") Bounds bounds) {
		if (bounds != null) res.getRequest().setAttribute("bounds", bounds);
		return new SuspendResponse.SuspendResponseBuilder<String>()
				.broadcaster(getBroadcaster())
				.outputComments(true)
				.addListener(listener)
				.build();
	}

	/**
	 * This URI allows a client to send a new Event that will be broadcaster
	 * to all other connected clients.
	 * @param event the Event (deserialized from JSON by Jersey)
	 * @return a Response
	 */
	@POST
	@Path("event")
	@Consumes(MediaType.APPLICATION_JSON)
	public Response broadcastEvent(Event event) {
		logger.info("New event: {}", event);
		getBroadcaster().broadcast(event);
		return Response.ok().build();
	}

	// ...

}
{% endhighlight %}

## Listen for client messages

In case of a duplex protocol, we said a client can send data back to the server using the connection. But how do we handle such messages? Atmosphere provides a **listener mechanism**, implementable via interfaces such as the [`AtmosphereResourceEventListener`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/AtmosphereResourceEventListener.html) or its WebSocket specialization, the [`WebSocketEventListener`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/websocket/WebSocketEventListener.html). The WebSocket interface exposes useful methods like `onConnect`, `onHandshake`, `onDisconnect`, or `onMessage`. Listeners are declared when the client connects (ie. when suspending a response).

{% highlight java %}
public class EventListener extends WebSocketEventListenerAdapter {

	private final Logger logger = LoggerFactory.getLogger(EventListener.class);

	@Override
	public void onMessage(WebSocketEvent event) {
		Bounds bounds = JsonUtils.fromJson(event.message(), Bounds.class);
		if (bounds == null) return;
		logger.info("New bounds {} for resource {}",
				event.message(), event.webSocket().resource().hashCode());
		AtmosphereRequest req = event.webSocket().resource().getRequest();
		req.setAttribute("bounds", bounds);
	}

}
{% endhighlight %}

## Filter your broadcasted messages

You may also have observed the `init()` method… but what is done inside exactly ?

A possibility offered by the framework is the ability to include logic before delivering message to each connected client. We can achieve that thanks to the [`BroadcastFilter`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/BroadcastFilter.html) interface and its specialization, the [`PerRequestBroadcastFilter`](http://atmosphere.github.com/atmosphere/apidocs/org/atmosphere/cpr/PerRequestBroadcastFilter.html) interface. The first interface allows to execute logic once (common to all clients) whereas the second one can apply to each client according to their associated context (session data, first connection headers, etc…).

Each BroadcastFilter can `CONTINUE` or `ABORT` a broadcast. While the broadcast is not aborted, the filter chain is executed in order (`BroadcastFilters` first, `PerRequestBroadcastFilters` then) and finally delivered (or not) to each client. The following snippet is an example of PerRequestBroadcastFilter implementation:

{% highlight java %}
public class BoundsFilter implements PerRequestBroadcastFilter {

    private final Logger logger = LoggerFactory.getLogger(BoundsFilter.class);

    @Override
    public BroadcastAction filter(Object originalMessage, Object message) {
        return new BroadcastAction(ACTION.CONTINUE, originalMessage);
    }

    @Override
    public BroadcastAction filter(AtmosphereResource res,
            Object originalMessage, Object message) {
        logger.info("BoundsFilter triggered for AtmosphereResource {} "+
				"with message {}", res.hashCode(), message);
        Event event = (Event) message;
        try {
            Bounds bounds = (Bounds) res.getRequest().getAttribute("bounds");
            if (bounds == null) throw new NoBoundsException("no bounds");
            if (bounds.contains(event)) {
                String json = JsonUtils.toJson(event); // Manual serialization
                return new BroadcastAction(ACTION.CONTINUE, json);
            } else {
                return new BroadcastAction(ACTION.ABORT, message);
            }
        } catch (NoBoundsException e) {
            logger.info("Applying default action CONTINUE, cause: {}",
					e.getMessage());
            String json = JsonUtils.toJson(event); // Manual serialization
            return new BroadcastAction(ACTION.CONTINUE, json);
        } catch (Exception e) {
            logger.info("Filter BoundsFilter aborted, cause: {}",
					e.getMessage());
            return new BroadcastAction(ACTION.ABORT, message);
        }
    }

}
{% endhighlight %}

# The client

Now that all is ready server side, we are able to connect our clients to the realtime endpoint. If you plan to use WebSocket, several clients are compatible with Atmosphere. Let’s focus on Java and Javascript ones:

- In java, you find several projects like [Java-WebSocket](https://github.com/TooTallNate/Java-WebSocket) or [async-http-client](https://github.com/sonatype/async-http-client). Some webapp containers also provide a WebSocket client implementation (e.g. [Jetty](http://download.eclipse.org/jetty/stable-7/apidocs/org/eclipse/jetty/websocket/package-summary.html))
- In Javascript, most of recent browsers implement the [WebSocket](http://dev.w3.org/html5/websockets/) interface. But, in case the browser is not compatible with WebSocket, this API can’t fallback into another protocol. Here comes the jQuery Atmosphere Plugin and its fallback mechanism.

In our case, you probably guessed, we’ll use the Atmosphere jQuery plugin. Please note that due to the jQuery dependency, we have to import the jQuery library in addition to the Atmosphere plugin.

Once done, connect to the server is no more complicated than the following `connect()` javascript routine:

{% highlight javascript %}
var endpoint;
function connect() {

    var callback = function callback(response) {
        // Websocket events.
        if (response.state == "opening") {
            console.log("Connected to realtime endpoint");
        } else if (response.state == "closed") {
            console.log("Disconnected from realtime endpoint");
        } else if (response.transport != 'polling' &&
                response.state == 'messageReceived') {
            if (response.status == 200) {
                var data = response.responseBody;
                if (data.length > 0) {
                    statsAgent.notify();
                    console.log("Message Received: " + data +
                            " & detected transport is " + response.transport);
                    var json = JSON.parse(data);
                    mapsAgent.drawEvent(json);
                }
            }
        }
    };

    var bounds = mapsAgent.getBounds();
    var header = bounds.southLat + "," + bounds.northLat +
            "," + bounds.westLng + "," + bounds.eastLng;
    endpoint = $.atmosphere.subscribe(url, callback, {
        transport: 'websocket',
		/* available transports: websocket, jsonp, long-polling,
			polling, streaming */
        attachHeadersAsQueryString: true,
        headers: {"X-Map-Bounds": header}
    });
}
{% endhighlight %}

You can see that we attach headers when connecting. They are used here to transmit some client context when connecting (actually, the current bounds of the map). As the WebSocket handshake doesn’t allow the client to pass any headers, they have to be serialized in the query string, justifying the `attachHeadersAsQueryString: true` entry. On the server side, the query string will be translated to headers so application wise, you don’t have to care about the difference.

To process message pushed from the server, we can add a callback. It will be triggered on each received message, passing a response (Javascript object) containing values as `status`, `state`, `transport` etc…

The `endpoint` variable – that stores the connection – was made global to be used when you want to disconnect the client from the realtime endpoint or push data to the server. The following snippet shows you the two corresponding javascript routines.

{% highlight javascript %}
function disconnect() {
	$.atmosphere.unsubscribe();
	endpoint = null;
}

function update(bounds) {
	console.log("### Map bounds changed:", JSON.stringify(bounds));
	if (!endpoint) return;
	endpoint.push(JSON.stringify(bounds));
}
{% endhighlight %}

# Testing your WebSocket resources

In addition to regular browser testing, you can easily try your realtime URIs with a shell and cURL:

{% highlight bash %}
curl -v -N -XGET http://localhost:8080/MapPush/api
{% endhighlight %}

- `-v/--verbose`: Make the operation more talkative
- `-N/--no-buffer`: Disable buffering of the output stream
- `-X/--request <command>`: Specify request command to use

With the second option, you will be able to observe all data sent by the server. Nonetheless, note that you’ll not be able to send data back to it.

cURL also allows you to send headers with the request:

{% highlight bash %}
boundsHeader='48.0,49.0,2.0,3.0'
curl -v -N -XGET http://localhost:8080/MapPush/api -H "X-MAP-BOUNDS: $boundsHeader"
{% endhighlight %}

- `-H/--header <line>`: Custom header to pass to server

You can also go deeper and analyse WebSocket frames with tools such as ngrep or Wireshark: these are a bit more complex tools but it may become very useful in some situations!

# Conclusion

Atmophere provides a powerful ecosystem that simplifies the creation of push applications and makes easy full-duplex communication between a server and any kind of client. Its intensive use of async I/O and its ready-to-use clustering plugins give it both performance and scalability. Moreover, the project is under Open source Apache license, the community is growing quickly and the last 0.9 version is stabilizing fast.

In short, the Atmosphere framework has a bright future :)

# Additional Resources

- Atmosphere project is hosted on GitHub:
  - Download the Atmosphere’s [whitepaper](https://github.com/Atmosphere/atmosphere/raw/master/docs/atmosphere_whitepaper.pdf)
  - Take a look at the [samples](https://github.com/Atmosphere/atmosphere/tree/master/samples)
  - Browse the [Javadoc](http://atmosphere.github.com/atmosphere/apidocs/)
- The community is reachable via Atmosphere’s [Google Group](http://groups.google.com/group/atmosphere-framework)
- You can also follow Atmosphere via Jean-Francois Arcand’s [blog](http://jfarcand.wordpress.com/) and [Twitter](http://twitter.com/atmo_framework)
