# fluent-plugin-http_shadow [![Build Status](https://secure.travis-ci.org/toyama0919/fluent-plugin-http_shadow.png?branch=master)](http://travis-ci.org/toyama0919/fluent-plugin-http_shadow)

copy http request. use shadow proxy server.

## Examples
```
<source>
  type tail
  format apache
  path /var/log/httpd/access_log
  pos_file /var/log/td-agent/access.pos
  tag apache.access
</source>

<match apache.access>
  type http_shadow
  host staging.exsample.com
  path_format ${path}
  method_key method
  header_hash { "Referer": "${referer}", "User-Agent": "${agent}" }
  body_key body
</match>
```

Assume following input is coming:

```
  {
    "host": "exsample.com",
    "ip_address": "127.0.0.1",
    "server": "10.0.0.11",
    "remote": "-",
    "time": "22/Dec/2014:03:20:26 +0900",
    "method": "GET",
    "path": "/hoge/?id=1",
    "code": "200",
    "size": "1578",
    "x_forwarded_proto": "http",
    "referer": "http://exsample.com/other/",
    "agent": "Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko"
    "body": "key=value"
  }
```

then result becomes as below (indented):

```
GET http://staging.exsample.com/hoge/?id=1
#=>  HTTP HEADER
#=>  "referer": "http://exsample.com/other/"
#=>  "agent": "Mozilla/5.0 (Windows NT 6.1; Trident/7.0; rv:11.0) like Gecko"
#=>  REQUEST BODY
#=>  "key=value"
```

## Examples(Virtual Host)
```
<match http_shadow.exsample>
  type http_shadow
  host_hash { 
    "www.example.com": "staging.example.com", 
    "api.example.com": "api-staging.example.com", 
    "blog.ipros.jp": "blog-staging.ipros.jp"
  }
  host_key host
  path_format ${path}
  method_key method
  protocol_format ${x_forwarded_proto} # default: http
  header_hash { "Referer": "${referer}", "User-Agent": "${user_agent}" }
  no_send_header_pattern ^(-|)$
</match>
```

## Examples(use cookie)
```
<match http_shadow.exsample>
  type http_shadow
  host_hash { 
    "www.example.com": "staging.example.com", 
    "api.example.com": "api-staging.example.com", 
    "blog.ipros.jp": "blog-staging.ipros.jp"
  }
  host_key host
  path_format ${path}
  method_key method
  header_hash { "Referer": "${referer}", "User-Agent": "${user_agent}" }
  cookie_hash {"rails-app_session": "${session_id}"}
</match>
```

## Examples(use rate_per_host_hash)
```
<match http_shadow.exsample>
  type http_shadow
  host_hash { 
    "www.example.com": "staging.example.com", 
    "api.example.com": "api-staging.example.com", 
    "blog.ipros.jp": "blog-staging.ipros.jp"
  }
  host_key host
  path_format ${path}
  method_key method
  header_hash { "Referer": "${referer}", "User-Agent": "${user_agent}" }
  rate_per_host_hash {
    "staging.example.com"     : 30,   # This means 30% requests to staging.example.com will be sent. Default(when not defined) value is 100.
    "api-staging.example.com" : 90
  }
</match>
```

## note

default GET Request.

## parameter

TODO

## todo

more test


## Installation
```
fluent-gem install fluent-plugin-http_shadow
```

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new [Pull Request](../../pull/new/master)

## Information

* [Homepage](https://github.com/toyama0919/fluent-plugin-http_shadow)
* [Issues](https://github.com/toyama0919/fluent-plugin-http_shadow/issues)
* [Documentation](http://rubydoc.info/gems/fluent-plugin-http_shadow/frames)
* [Email](mailto:toyama0919@gmail.com)

## Copyright

Copyright (c) 2015 Hiroshi Toyama

