## The Valve Component

### Introduction

A **Valve** element represents a component that will be inserted into the request processing pipeline for the associated Catalina container ([Engine](https://tomcat.apache.org/tomcat-8.0-doc/config/engine.html), [Host](https://tomcat.apache.org/tomcat-8.0-doc/config/host.html), or [Context](https://tomcat.apache.org/tomcat-8.0-doc/config/context.html)). Individual Valves have distinct processing capabilities, and are described individually below.

*The description below uses the variable name `$CATALINA_BASE` to refer the base directory against which most relative paths are resolved. If you have not configured Tomcat for multiple instances by setting a `CATALINA_BASE` directory, then `$CATALINA_BASE` will be set to the value of `$CATALINA_HOME`, the directory into which you have installed Tomcat.*

### Access Logging

Access logging is performed by valves that implement **org.apache.catalina.AccessLog** interface.

#### Access Log Valve

#### Introduction

The **Access Log Valve** creates log files in the same format as those created by standard web servers. These logs can later be analyzed by standard log analysis tools to track page hit counts, user session activity, and so on. This `Valve` uses self-contained logic to write its log files, which can be automatically rolled over at midnight each day. (The essential requirement for access logging is to handle a large continuous stream of data with low overhead. This `Valve` does not use Apache Commons Logging, thus avoiding additional overhead and potentially complex configuration).

This `Valve` may be associated with any Catalina container (`Context`, `Host`, or `Engine`), and will record ALL requests processed by that container.

Some requests may be handled by Tomcat before they are passed to a container. These include redirects from /foo to /foo/ and the rejection of invalid requests. Where Tomcat can identify the `Context` that would have handled the request, the request/response will be logged in the `AccessLog`(s) associated `Context`, `Host` and `Engine`. Where Tomcat cannot identify the `Context` that would have handled the request, e.g. in cases where the URL is invalid, Tomcat will look first in the `Engine`, then the default `Host` for the `Engine` and finally the ROOT (or default) `Context` for the default `Host` for an `AccessLog` implementation. Tomcat will use the first `AccessLog` implementation found to log those requests that are rejected before they are passed to a container.

The output file will be placed in the directory given by the `directory` attribute. The name of the file is composed by concatenation of the configured `prefix`, timestamp and `suffix`. The format of the timestamp in the file name can be set using the `fileDateFormat` attribute. This timestamp will be omitted if the file rotation is switched off by setting `rotatable` to `false`.

**Warning:** If multiple AccessLogValve instances are used, they should be configured to use different output files.

If sendfile is used, the response bytes will be written asynchronously in a separate thread and the access log valve will not know how many bytes were actually written. In this case, the number of bytes that was passed to the sendfile thread for writing will be recorded in the access log valve.

#### Attributes

The **Access Log Valve** supports the following configuration attributes:

| Attribute                  | Description                              |
| -------------------------- | ---------------------------------------- |
| **className**              | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.AccessLogValve** to use the default access log valve. |
| `directory`                | Absolute or relative pathname of a directory in which log files created by this valve will be placed. If a relative path is specified, it is interpreted as relative to $CATALINA_BASE. If no directory attribute is specified, the default value is "logs" (relative to $CATALINA_BASE). |
| `prefix`                   | The prefix added to the start of each log file's name. If not specified, the default value is "access_log". |
| `suffix`                   | The suffix added to the end of each log file's name. If not specified, the default value is "" (a zero-length string), meaning that no suffix will be added. |
| `fileDateFormat`           | Allows a customized timestamp in the access log file name. The file is rotated whenever the formatted timestamp changes. The default value is `.yyyy-MM-dd`. If you wish to rotate every hour, then set this value to `.yyyy-MM-dd.HH`. The date format will always be localized using the locale `en_US`. |
| `rotatable`                | Flag to determine if log rotation should occur. If set to `false`, then this file is never rotated and `fileDateFormat` is ignored. Default value: `true` |
| `renameOnRotate`           | By default for a rotatable log the active access log file name will contain the current timestamp in `fileDateFormat`. During rotation the file is closed and a new file with the next timestamp in the name is created and used. When setting `renameOnRotate` to `true`, the timestamp is no longer part of the active log file name. Only during rotation the file is closed and then renamed to include the timestamp. This is similar to the behavior of most log frameworks when doing time based rotation. Default value: `false` |
| `pattern`                  | A formatting layout identifying the various information fields from the request and response to be logged, or the word `common` or `combined` to select a standard format. See below for more information on configuring this attribute. |
| `encoding`                 | Character set used to write the log file. An empty string means to use the system default character set. Default value: use the system default character set. |
| `locale`                   | The locale used to format timestamps in the access log lines. Any timestamps configured using an explicit SimpleDateFormat pattern (`%{xxx}t`) are formatted in this locale. By default the default locale of the Java process is used. Switching the locale after the AccessLogValve is initialized is not supported. Any timestamps using the common log format (`CLF`) are always formatted in the locale `en_US`. |
| `requestAttributesEnabled` | Set to `true` to check for the existence of request attributes (typically set by the RemoteIpValve and similar) that should be used to override the values returned by the request for remote address, remote host, server port and protocol. If the attributes are not set, or this attribute is set to `false` then the values from the request will be used. If not set, the default value of `false` will be used. |
| `conditionIf`              | Turns on conditional logging. If set, requests will be logged only if `ServletRequest.getAttribute()` is not null. For example, if this value is set to `important`, then a particular request will only be logged if `ServletRequest.getAttribute("important") != null`. The use of Filters is an easy way to set/unset the attribute in the ServletRequest on many different requests. |
| `conditionUnless`          | Turns on conditional logging. If set, requests will be logged only if `ServletRequest.getAttribute()` is null. For example, if this value is set to `junk`, then a particular request will only be logged if `ServletRequest.getAttribute("junk") == null`. The use of Filters is an easy way to set/unset the attribute in the ServletRequest on many different requests. |
| `condition`                | The same as `conditionUnless`. This attribute is provided for backwards compatibility. |
| `buffered`                 | Flag to determine if logging will be buffered. If set to `false`, then access logging will be written after each request. Default value: `true` |
| `maxLogMessageBufferSize`  | Log message buffers are usually recycled and re-used. To prevent excessive memory usage, if a buffer grows beyond this size it will be discarded. The default is `256` characters. This should be set to larger than the typical access log message size. |
| `resolveHosts`             | This attribute is no longer supported. Use the connector attribute `enableLookups` instead.If you have `enableLookups` on the connector set to `true` and want to ignore it, use **%a** instead of **%h** in the value of `pattern`. |

Values for the `pattern` attribute are made up of literal text strings, combined with pattern identifiers prefixed by the "%" character to cause replacement by the corresponding variable value from the current request and response. The following pattern codes are supported:

- **%a** - Remote IP address
- **%A** - Local IP address
- **%b** - Bytes sent, excluding HTTP headers, or '-' if zero
- **%B** - Bytes sent, excluding HTTP headers
- **%h** - Remote host name (or IP address if `enableLookups` for the connector is false)
- **%H** - Request protocol
- **%l** - Remote logical username from identd (always returns '-')
- **%m** - Request method (GET, POST, etc.)
- **%p** - Local port on which this request was received. See also `%{xxx}p` below.
- **%q** - Query string (prepended with a '?' if it exists)
- **%r** - First line of the request (method and request URI)
- **%s** - HTTP status code of the response
- **%S** - User session ID
- **%t** - Date and time, in Common Log Format
- **%u** - Remote user that was authenticated (if any), else '-'
- **%U** - Requested URL path
- **%v** - Local server name
- **%D** - Time taken to process the request, in millis
- **%T** - Time taken to process the request, in seconds
- **%F** - Time taken to commit the response, in millis
- **%I** - Current request thread name (can compare later with stacktraces)

There is also support to write information incoming or outgoing headers, cookies, session or request attributes and special timestamp formats. It is modeled after the [Apache HTTP Server](http://httpd.apache.org/) log configuration syntax. Each of them can be used multiple times with different `xxx` keys:

- **%{xxx}i** write value of incoming header with name `xxx`
- **%{xxx}o** write value of outgoing header with name `xxx`
- **%{xxx}c** write value of cookie with name `xxx`
- **%{xxx}r** write value of ServletRequest attribute with name `xxx`
- **%{xxx}s** write value of HttpSession attribute with name `xxx`
- **%{xxx}p** write local (server) port (`xxx==local`) or remote (client) port (`xxx=remote`)
- **%{xxx}t** write timestamp at the end of the request formatted using the enhanced SimpleDateFormat pattern `xxx`

All formats supported by SimpleDateFormat are allowed in `%{xxx}t`. In addition the following extensions have been added:

- **sec** - number of seconds since the epoch
- **msec** - number of milliseconds since the epoch
- **msec_frac** - millisecond fraction

These formats can not be mixed with SimpleDateFormat formats in the same format token.

Furthermore one can define whether to log the timestamp for the request start time or the response finish time:

- **begin** or prefix **begin:** chooses the request start time
- **end** or prefix **end:** chooses the response finish time

By adding multiple `%{xxx}t` tokens to the pattern, one can also log both timestamps.

The shorthand pattern `pattern="common"` corresponds to the Common Log Format defined by **'%h %l %u %t "%r" %s %b'**.

The shorthand pattern `pattern="combined"` appends the values of the `Referer` and `User-Agent` headers, each in double quotes, to the `common` pattern.

When Tomcat is operating behind a reverse proxy, the client information logged by the Access Log Valve may represent the reverse proxy, the browser or some combination of the two depending on the configuration of Tomcat and the reverse proxy. For Tomcat configuration options see [Proxies Support](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Proxies_Support) and the [Proxy How-To](https://tomcat.apache.org/tomcat-8.0-doc/proxy-howto.html). For reverse proxies that use mod_jk, see the [generic proxy](http://tomcat.apache.org/connectors-doc/generic_howto/proxy.html)documentation. For other reverse proxies, consult their documentation.

#### Extended Access Log Valve

#### Introduction

The **Extended Access Log Valve** extends the [Access Log Valve](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Access_Log_Valve) class, and so uses the same self-contained logging logic. This means it implements many of the same file handling attributes. The main difference to the standard `AccessLogValve` is that `ExtendedAccessLogValve` creates log files which conform to the Working Draft for the [Extended Log File Format](http://www.w3.org/TR/WD-logfile.html) defined by the W3C.

#### Attributes

The **Extended Access Log Valve** supports all configuration attributes of the standard [Access Log Valve.](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Access_Log_Valve) Only the values used for `className` and `pattern` differ.

| Attribute     | Description                              |
| ------------- | ---------------------------------------- |
| **className** | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.ExtendedAccessLogValve** to use the extended access log valve. |
| `pattern`     | A formatting layout identifying the various information fields from the request and response to be logged. See below for more information on configuring this attribute. |

Values for the `pattern` attribute are made up of format tokens. Some of the tokens need an additional prefix. Possible prefixes are `c` for "client", `s` for "server", `cs` for "client to server", `sc` for "server to client" or `x` for "application specific". Furthermore some tokens are completed by an additional selector. See the [W3C specification](http://www.w3.org/TR/WD-logfile.html) for more information about the format.

The following format tokens are supported:

- **bytes** - Bytes sent, excluding HTTP headers, or '-' if zero
- **c-dns** - Remote host name (or IP address if `enableLookups` for the connector is false)
- **c-ip** - Remote IP address
- **cs-method** - Request method (GET, POST, etc.)
- **cs-uri** - Request URI
- **cs-uri-query** - Query string (prepended with a '?' if it exists)
- **cs-uri-stem** - Requested URL path
- **date** - The date in yyyy-mm-dd format for GMT
- **s-dns** - Local host name
- **s-ip** - Local IP address
- **sc-status** - HTTP status code of the response
- **time** - Time the request was served in HH:mm:ss format for GMT
- **time-taken** - Time (in seconds as floating point) taken to serve the request
- **x-threadname** - Current request thread name (can compare later with stacktraces)

For any of the `x-H(XXX)` the following method will be called from the HttpServletRequest object:

- **x-H(authType)**: getAuthType
- **x-H(characterEncoding)**: getCharacterEncoding
- **x-H(contentLength)**: getContentLength
- **x-H(locale)**: getLocale
- **x-H(protocol)**: getProtocol
- **x-H(remoteUser)**: getRemoteUser
- **x-H(requestedSessionId)**: getRequestedSessionId
- **x-H(requestedSessionIdFromCookie)**: isRequestedSessionIdFromCookie
- **x-H(requestedSessionIdValid)**: isRequestedSessionIdValid
- **x-H(scheme)**: getScheme
- **x-H(secure)**: isSecure

There is also support to write information about headers cookies, context, request or session attributes and request parameters.

- **cs(XXX)** for incoming request headers with name XXX
- **sc(XXX)** for outgoing response headers with name XXX
- **x-A(XXX)** for the servlet context attribute with name XXX
- **x-C(XXX)** for the first cookie with name XXX
- **x-O(XXX)** for a concatenation of all outgoing response headers with name XXX
- **x-P(XXX)** for the URL encoded (using UTF-8) request parameter with name XXX
- **x-R(XXX)** for the request attribute with name XXX
- **x-S(XXX)** for the session attribute with name XXX

### Access Control

#### Remote Address Filter

#### Introduction

The **Remote Address Filter** allows you to compare the IP address of the client that submitted this request against one or more *regular expressions*, and either allow the request to continue or refuse to process the request from this client. A Remote Address Filter can be associated with any Catalina container ([Engine](https://tomcat.apache.org/tomcat-8.0-doc/config/engine.html), [Host](https://tomcat.apache.org/tomcat-8.0-doc/config/host.html), or [Context](https://tomcat.apache.org/tomcat-8.0-doc/config/context.html)), and must accept any request presented to this container for processing before it will be passed on.

The syntax for *regular expressions* is different than that for 'standard' wildcard matching. Tomcat uses the `java.util.regex` package. Please consult the Java documentation for details of the expressions supported.

Optionally one can append the server connector port separated with a semicolon (";") to allow different expressions for each connector.

The behavior when a request is refused can be changed to not deny but instead set an invalid `authentication` header. This is useful in combination with the context attribute`preemptiveAuthentication="true"`.

**Note:** There is a caveat when using this valve with IPv6 addresses. Format of the IP address that this valve is processing depends on the API that was used to obtain it. If the address was obtained from Java socket using Inet6Address class, its format will be `x:x:x:x:x:x:x:x`. That is, the IP address for localhost will be `0:0:0:0:0:0:0:1` instead of the more widely used `::1`. Consult your access logs for the actual value.

See also: [Remote Host Filter](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Remote_Host_Filter), [Remote IP Valve](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Remote_IP_Valve).

#### Attributes

The **Remote Address Filter** supports the following configuration attributes:

| Attribute                       | Description                              |
| ------------------------------- | ---------------------------------------- |
| **className**                   | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.RemoteAddrValve**. |
| `allow`                         | A regular expression (using `java.util.regex`) that the remote client's IP address is compared to. If this attribute is specified, the remote address MUST match for this request to be accepted. If this attribute is not specified, all requests will be accepted UNLESS the remote address matches a `deny` pattern. |
| `deny`                          | A regular expression (using `java.util.regex`) that the remote client's IP address is compared to. If this attribute is specified, the remote address MUST NOT match for this request to be accepted. If this attribute is not specified, request acceptance is governed solely by the `allow` attribute. |
| `denyStatus`                    | HTTP response status code that is used when rejecting denied request. The default value is `403`. For example, it can be set to the value `404`. |
| `addConnectorPort`              | Append the server connector port to the client IP address separated with a semicolon (";"). If this is set to `true`, the expressions configured with `allow` and`deny` is compared against `ADDRESS;PORT` where `ADDRESS` is the client IP address and `PORT` is the Tomcat connector port which received the request. The default value is `false`. |
| `invalidAuthenticationWhenDeny` | When a request should be denied, do not deny but instead set an invalid `authentication` header. This only works if the context has the attribute `preemptiveAuthentication="true"` set. An already existing `authentication` header will not be overwritten. In effect this will trigger authentication instead of deny even if the application does not have a security constraint configured.This can be combined with `addConnectorPort` to trigger authentication depending on the client and the connector that is used to access an application. |

#### Example 1

To allow access only for the clients connecting from localhost:

```
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
   allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1"/>
```

#### Example 2

To allow unrestricted access for the clients connecting from localhost but for all other clients only to port 8443:

```
<Valve className="org.apache.catalina.valves.RemoteAddrValve"
   addConnectorPort="true"
   allow="127\.\d+\.\d+\.\d+;\d*|::1;\d*|0:0:0:0:0:0:0:1;\d*|.*;8443"/>
```

#### Example 3

To allow unrestricted access to port 8009, but trigger basic authentication if the application is accessed on another port:

```
<Context>
  ...
  <Valve className="org.apache.catalina.valves.RemoteAddrValve"
         addConnectorPort="true"
         invalidAuthenticationWhenDeny="true"
         allow=".*;8009"/>
  <Valve className="org.apache.catalina.authenticator.BasicAuthenticator" />
  ...
</Context>
```

#### Remote Host Filter

#### Introduction

The **Remote Host Filter** allows you to compare the hostname of the client that submitted this request against one or more *regular expressions*, and either allow the request to continue or refuse to process the request from this client. A Remote Host Filter can be associated with any Catalina container ([Engine](https://tomcat.apache.org/tomcat-8.0-doc/config/engine.html), [Host](https://tomcat.apache.org/tomcat-8.0-doc/config/host.html), or [Context](https://tomcat.apache.org/tomcat-8.0-doc/config/context.html)), and must accept any request presented to this container for processing before it will be passed on.

The syntax for *regular expressions* is different than that for 'standard' wildcard matching. Tomcat uses the `java.util.regex` package. Please consult the Java documentation for details of the expressions supported.

Optionally one can append the server connector port separated with a semicolon (";") to allow different expressions for each connector.

The behavior when a request is refused can be changed to not deny but instead set an invalid `authentication` header. This is useful in combination with the context attribute`preemptiveAuthentication="true"`.

**Note:** This filter processes the value returned by method `ServletRequest.getRemoteHost()`. To allow the method to return proper host names, you have to enable "DNS lookups" feature on a **Connector**.

See also: [Remote Address Filter](https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html#Remote_Address_Filter), [HTTP Connector](https://tomcat.apache.org/tomcat-8.0-doc/config/http.html) configuration.

#### Attributes

The **Remote Host Filter** supports the following configuration attributes:

| Attribute                       | Description                              |
| ------------------------------- | ---------------------------------------- |
| **className**                   | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.RemoteHostValve**. |
| `allow`                         | A regular expression (using `java.util.regex`) that the remote client's hostname is compared to. If this attribute is specified, the remote hostname MUST match for this request to be accepted. If this attribute is not specified, all requests will be accepted UNLESS the remote hostname matches a `deny` pattern. |
| `deny`                          | A regular expression (using `java.util.regex`) that the remote client's hostname is compared to. If this attribute is specified, the remote hostname MUST NOT match for this request to be accepted. If this attribute is not specified, request acceptance is governed solely by the `allow` attribute. |
| `denyStatus`                    | HTTP response status code that is used when rejecting denied request. The default value is `403`. For example, it can be set to the value `404`. |
| `addConnectorPort`              | Append the server connector port to the client hostname separated with a semicolon (";"). If this is set to `true`, the expressions configured with `allow` and`deny` is compared against `HOSTNAME;PORT` where `HOSTNAME` is the client hostname and `PORT` is the Tomcat connector port which received the request. The default value is `false`. |
| `invalidAuthenticationWhenDeny` | When a request should be denied, do not deny but instead set an invalid `authentication` header. This only works if the context has the attribute `preemptiveAuthentication="true"` set. An already existing `authentication` header will not be overwritten. In effect this will trigger authentication instead of deny even if the application does not have a security constraint configured.This can be combined with `addConnectorPort` to trigger authentication depending on the client and the connector that is used to access an application. |

### Proxies Support

#### Remote IP Valve

#### Introduction

Tomcat port of [mod_remoteip](http://httpd.apache.org/docs/trunk/mod/mod_remoteip.html), this valve replaces the apparent client remote IP address and hostname for the request with the IP address list presented by a proxy or a load balancer via a request headers (e.g. "X-Forwarded-For").

Another feature of this valve is to replace the apparent scheme (http/https), server port and `request.secure` with the scheme presented by a proxy or a load balancer via a request header (e.g. "X-Forwarded-Proto").

This Valve may be used at the `Engine`, `Host` or `Context` level as required. Normally, this Valve would be used at the `Engine` level.

If used in conjunction with Remote Address/Host valves then this valve should be defined first to ensure that the correct client IP address is presented to the Remote Address/Host valves.

**Note:** By default this valve has no effect on the values that are written into access log. The original values are restored when request processing leaves the valve and that always happens earlier than access logging. To pass the remote address, remote host, server port and protocol values set by this valve to the access log, they are put into request attributes. Publishing these values here is enabled by default, but `AccessLogValve` should be explicitly configured to use them. See documentation for `requestAttributesEnabled` attribute of `AccessLogValve`.

The names of request attributes that are set by this valve and can be used by access logging are the following:

- `org.apache.catalina.AccessLog.RemoteAddr`
- `org.apache.catalina.AccessLog.RemoteHost`
- `org.apache.catalina.AccessLog.Protocol`
- `org.apache.catalina.AccessLog.ServerPort`
- `org.apache.tomcat.remoteAddr`

#### Attributes

The **Remote IP Valve** supports the following configuration attributes:

| Attribute                  | Description                              |
| -------------------------- | ---------------------------------------- |
| **className**              | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.RemoteIpValve**. |
| `remoteIpHeader`           | Name of the HTTP Header read by this valve that holds the list of traversed IP addresses starting from the requesting client. If not specified, the default of `x-forwarded-for` is used. |
| `internalProxies`          | Regular expression (using `java.util.regex`) that a proxy's IP address must match to be considered an internal proxy. Internal proxies that appear in the **remoteIpHeader** will be trusted and will not appear in the **proxiesHeader** value. If not specified the default value of `10\.\d{1,3}\.\d{1,3}\.\d{1,3}|192\.168\.\d{1,3}\.\d{1,3}|169\.254\.\d{1,3}\.\d{1,3}|127\.\d{1,3}\.\d{1,3}\.\d{1,3}|172\.1[6-9]{1}\.\d{1,3}\.\d{1,3}|172\.2[0-9]{1}\.\d{1,3}\.\d{1,3}|172\.3[0-1]{1}\.\d{1,3}\.\d{1,3} `will be used. |
| `proxiesHeader`            | Name of the HTTP header created by this valve to hold the list of proxies that have been processed in the incoming **remoteIpHeader**. If not specified, the default of `x-forwarded-by` is used. |
| `requestAttributesEnabled` | Set to `true` to set the request attributes used by AccessLog implementations to override the values returned by the request for remote address, remote host, server port and protocol. Request attributes are also used to enable the forwarded remote address to be displayed on the status page of the Manager web application. If not set, the default value of `true` will be used. |
| `trustedProxies`           | Regular expression (using `java.util.regex`) that a proxy's IP address must match to be considered an trusted proxy. Trusted proxies that appear in the **remoteIpHeader** will be trusted and will appear in the **proxiesHeader** value. If not specified, no proxies will be trusted. |
| `protocolHeader`           | Name of the HTTP Header read by this valve that holds the protocol used by the client to connect to the proxy. If not specified, the default of `null` is used. |
| `portHeader`               | Name of the HTTP Header read by this valve that holds the port used by the client to connect to the proxy. If not specified, the default of `null` is used. |
| `protocolHeaderHttpsValue` | Value of the **protocolHeader** to indicate that it is an HTTPS request. If not specified, the default of `https` is used. |
| `httpServerPort`           | Value returned by `ServletRequest.getServerPort()` when the **protocolHeader** indicates `http` protocol and no **portHeader** is present. If not specified, the default of `80` is used. |
| `httpsServerPort`          | Value returned by `ServletRequest.getServerPort()` when the **protocolHeader** indicates `https` protocol and no **portHeader** is present. If not specified, the default of `443` is used. |
| `changeLocalPort`          | If `true`, the value returned by `ServletRequest.getLocalPort()` and `ServletRequest.getServerPort()` is modified by the this valve. If not specified, the default of `false` is used. |

#### SSL Valve

#### Introduction

When using mod_proxy_http, the client SSL information is not included in the protocol (unlike mod_jk and mod_proxy_ajp). To make the client SSL information available to Tomcat, some additional configuration is required. In httpd, mod_headers is used to add the SSL information as HTTP headers. In Tomcat, this valve is used to read the information from the HTTP headers and insert it into the request.

Note: Ensure that the headers are always set by httpd for all requests to prevent a client spoofing SSL information by sending fake headers.

To configure httpd to set the necessary headers, add the following:

```
<IfModule ssl_module>
  RequestHeader set SSL_CLIENT_CERT "%{SSL_CLIENT_CERT}s"
  RequestHeader set SSL_CIPHER "%{SSL_CIPHER}s"
  RequestHeader set SSL_SESSION_ID "%{SSL_SESSION_ID}s"
  RequestHeader set SSL_CIPHER_USEKEYSIZE "%{SSL_CIPHER_USEKEYSIZE}s"
</IfModule>
```

#### Attributes

The **SSL Valve** supports the following configuration attribute:

| Attribute                    | Description                              |
| ---------------------------- | ---------------------------------------- |
| **className**                | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.SSLValve**. |
| `sslClientCertHeader`        | Allows setting a custom name for the ssl_client_cert header. If not specified, the default of `ssl_client_cert` is used. |
| `sslCipherHeader`            | Allows setting a custom name for the ssl_cipher header. If not specified, the default of `ssl_cipher` is used. |
| `sslSessionIdHeader`         | Allows setting a custom name for the ssl_session_id header. If not specified, the default of `ssl_session_id` is used. |
| `sslCipherUserKeySizeHeader` | Allows setting a custom name for the ssl_cipher_usekeysize header. If not specified, the default of `ssl_cipher_usekeysize` is used. |

### Single Sign On Valve

#### Introduction

The *Single Sign On Valve* is utilized when you wish to give users the ability to sign on to any one of the web applications associated with your virtual host, and then have their identity recognized by all other web applications on the same virtual host.

See the [Single Sign On](https://tomcat.apache.org/tomcat-8.0-doc/config/host.html#Single_Sign_On) special feature on the **Host** element for more information.

#### Attributes

The **Single Sign On** Valve supports the following configuration attributes:

| Attribute                 | Description                              |
| ------------------------- | ---------------------------------------- |
| **className**             | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.authenticator.SingleSignOn**. |
| `requireReauthentication` | Default false. Flag to determine whether each request needs to be reauthenticated to the security **Realm**. If "true", this Valve uses cached security credentials (username and password) to reauthenticate to the **Realm** each request associated with an SSO session. If "false", the Valve can itself authenticate requests based on the presence of a valid SSO cookie, without rechecking with the **Realm**. |
| `cookieDomain`            | Sets the host domain to be used for sso cookies. |

### Error Report Valve

#### Introduction

The **Error Report Valve** is a simple error handler for HTTP status codes that will generate and return HTML error pages.

**NOTE:** Disabling both showServerInfo and showReport will only return the HTTP status code and remove all CSS.

#### Attributes

The **Error Report Valve** supports the following configuration attributes:

| Attribute        | Description                              |
| ---------------- | ---------------------------------------- |
| **className**    | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.ErrorReportValve** to use the default error report valve. |
| `showReport`     | Flag to determine if the error report is presented when an error occurs. If set to `false`, then the error report is not in the HTML response. Default value: `true` |
| `showServerInfo` | Flag to determine if server information is presented when an error occurs. If set to `false`, then the server version is not returned in the HTML response. Default value: `true` |

### Crawler Session Manager Valve

#### Introduction

Web crawlers can trigger the creation of many thousands of sessions as they crawl a site which may result in significant memory consumption. This Valve ensures that crawlers are associated with a single session - just like normal users - regardless of whether or not they provide a session token with their requests.

This Valve may be used at the `Engine`, `Host` or `Context` level as required. Normally, this Valve would be used at the `Engine` level.

If used in conjunction with Remote IP valve then the Remote IP valve should be defined before this valve to ensure that the correct client IP address is presented to this valve.

#### Attributes

The **Crawler Session Manager Valve** supports the following configuration attributes:

| Attribute                 | Description                              |
| ------------------------- | ---------------------------------------- |
| **className**             | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.CrawlerSessionManagerValve**. |
| `crawlerIps`              | Regular expression (using `java.util.regex`) that client IP is matched against to determine if a request is from a web crawler. By default such regular expression is not set. |
| `crawlerUserAgents`       | Regular expression (using `java.util.regex`) that the user agent HTTP request header is matched against to determine if a request is from a web crawler. If not set, the default of `.*[bB]ot.*|.*Yahoo! Slurp.*|.*Feedfetcher-Google.*` is used. |
| `sessionInactiveInterval` | The minimum time in seconds that the Crawler Session Manager Valve should keep the mapping of client IP to session ID in memory without any activity from the client. The client IP / session cache will be periodically purged of mappings that have been inactive for longer than this interval. If not specified the default value of `60`will be used. |

### Stuck Thread Detection Valve

#### Introduction

This valve allows to detect requests that take a long time to process, which might indicate that the thread that is processing it is stuck. Additionally it can optionally interrupt such threads to try and unblock them.

When such a request is detected, the current stack trace of its thread is written to Tomcat log with a WARN level.

The IDs and names of the stuck threads are available through JMX in the `stuckThreadIds` and `stuckThreadNames` attributes. The IDs can be used with the standard Threading JVM MBean (`java.lang:type=Threading`) to retrieve other information about each stuck thread.

#### Attributes

The **Stuck Thread Detection Valve** supports the following configuration attributes:

| Attribute                  | Description                              |
| -------------------------- | ---------------------------------------- |
| **className**              | Java class name of the implementation to use. This MUST be set to **org.apache.catalina.valves.StuckThreadDetectionValve**. |
| `threshold`                | Minimum duration in seconds after which a thread is considered stuck. Default is 600 seconds. If set to 0, the detection is disabled.Note: since the detection (and optional interruption) is done in the background thread of the Container (Engine, Host or Context) declaring this Valve, the threshold should be higher than the `backgroundProcessorDelay` of this Container. |
| `interruptThreadThreshold` | Minimum duration in seconds after which a stuck thread should be interrupted to attempt to "free" it.Note that there's no guarantee that the thread will get unstuck. This usually works well for threads stuck on I/O or locks, but is probably useless in case of infinite loops.Default is -1 which disables the feature. To enable it, the value must be greater or equal to `threshold`. |

### Semaphore Valve

#### Introduction

The **Semaphore Valve** is able to limit the number of concurrent request processing threads.

**org.apache.catalina.valves.SemaphoreValve** provides methods which may be overridden by a subclass to customize behavior:

- **controlConcurrency** may be overridden to add conditions;
- **permitDenied** may be overridden to add error handling when a permit isn't granted.

#### Attributes

The **Semaphore Valve** supports the following configuration attributes:

| Attribute       | Description                              |
| --------------- | ---------------------------------------- |
| `block`         | Flag to determine if a thread is blocked until a permit is available. The default value is **true**. |
| **className**   | Java class name of the implementation to use. This MUST be set to**org.apache.catalina.valves.SemaphoreValve**. |
| `concurrency`   | Concurrency level of the semaphore. The default value is **10**. |
| `fairness`      | Fairness of the semaphore. The default value is **false**. |
| `interruptible` | Flag to determine if a thread may be interrupted until a permit is available. The default value is **false**. |



参考:  <https://www.oxxus.net/tutorials/tomcat/tomcat-valve>

原文链接: <https://tomcat.apache.org/tomcat-8.0-doc/config/valve.html>




