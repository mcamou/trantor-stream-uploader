Stream Uploader Proxy Servlet
-----------------------------
Stream-uploading big files with Ruby Rack 1.x [is impossible](https://groups.google.com/forum/?fromgroups=#!topic/rack-devel/T5YE-aFzSIQ).
The Problem is that [Rack would read the entire request body into memory](http://stackoverflow.com/questions/3027564), and there
are no obvious ways to interact with raw requests and responses in this framework. AFAIK, to accomplish this task, people
just use non-Rack stuff, such as [Goliath](https://github.com/postrank-labs/goliath), Node.js, JavaEE or even the venerable FastCGI.

As a JRuby-friendly way, this project consists of simple Servlet, packaged in a WAR file ready to be deployed in
[Torquebox](http://torquebox.org/documentation/). The stream is guaranteed to be 1KB wide, so its memory footprint is
quite ridiculous. And it's fast.

In my case, the motivation came from Trantor, an internal doc archiving system we are developing. Trantor is distributed,
so the web front-end and API (Sinatra/Rack app) lives in a different server than the file server back-end (another Sinatra/Rack app).
The front-end app also hosts a `FileServiceProxy < Sinatra::Base` proxy for the GET, DELETE, OPTIONS and HEAD XMLHttpRequest:s
coming from the HTML5 client, which uses [jQuery-File-Upload](https://github.com/blueimp/jQuery-File-Upload). Because of
the limitations of Rack mentioned above, the PUT/POST requests (file creation and versioning) must be routed through the Servlet.

### Configure it
Its configuration is done via `src/main/webapp/web.xml` is simple:

```xml
...
  <servlet>
    ...
    <init-param>
      <param-name>targetUri</param-name>
      <param-value>http://remotehost:9090/files</param-value>
    </init-param>
  </servlet>
...
```

The resulting WAR will be deployed at the `/uploader` context. If you wish to change it, then edit `src/main/webapp/jboss-web.xml`.

### Build it
This is a Maven 3 project. Mosey along: `mvn clean package`

### Deploy it (JBoss 7 AS)
After the build process is complete, drop `target/stream-uploader.war` into `$JBOSS_HOME/standalone/deployments`

### TO-DO
* More tests
* Implement authorization and authentication API


### License
Licensed under the Apache License, Version 2.0. See LICENSE file for more details.

### Credits
The inspiration came from (this StackOverflow post)[http://stackoverflow.com/questions/2471799] and I stole good implementation
 ideas from the (HTTP-Proxy-Servlet)[https://github.com/dsmiley/HTTP-Proxy-Servlet] GitHub project by dsmiley.
