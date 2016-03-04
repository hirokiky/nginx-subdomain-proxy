# nginx-subdomain-proxy

Nginx container to pass requests to another host with same sub domain.

If you specify

* example.local as "from host"
* example.com as "to host"

This nginx container will pass all requests like this

```
*.example.local => *.example.com
```

## Why nginx-subdomain-proxy

It's useful to create proxy server for inner Web services (like `*.mydomain.private`).
If you run inner services in AWS VPC and want to access from out of the VPC for debugging,
Just you need launch this container with `*.mydomain.com` as "from host" and `*mydomain.private` as "to host".

```
                          VPC
                          +-------------------------------------------+
                          |                                           |
[Web] *.mydomain.com--> proxy -> *.mydomain.private [inner services]  |
                          |                                           |
                          +-------------------------------------------+

```

## How to use

You can run this container without inheriting Dockerfile, just specify these arguments.

* `FROM_HOST_REGEXP`: A regexp for "from host" like (`mydomain\.com`)
* `TO_HOST`: string for "to host" (`mydomain.private`)
    * or `PROXY_PASS`: string for `proxy_pass`. You can use `\$subdomain` variable (The back slash is necessary) like `http://foo-\$subdomain.mydomain.private`. If you specified `TO_HOST`, don't specify this.
* `RESOLVER_IP`: IP address for DNS server. If you use privete Route53 within AWS VPC, specify `*.*.*.2` value in your VPC IP range.
* `RESOLVER_VALID_SEC`: Integer for available seconds of caching DNS records. By default it will cache forever.
* `X_FORWARDED_PROTO`: Value passed by X_FORWARDED_PROTO header (If `X-Forwarded-Proto` was not specified by client).
* `IGNORE_HTTP`: If specified `IGNORE_HTTP=1` and `X-Forwarded-Proto` was `http` , the server will redirect to https://...
* `LIMIT_NON_AUTHZ`: Set `1` if you want to limit requests without Authorization Header by X-Forwarded-For
    * `LIMIT_RATE`: Set value for the `rate` argument of `limit_req_zone`. The default value is '1000r/s'. See http://nginx.org/en/docs/http/ngx_http_limit_req_module.html

and you need to mount `/var/run/docker.sock` at `/tmp/docker.sock` in the container.

## Example

```
sudo docker run -e "FROM_HOST_REGEXP=mydomain\.com" -e "TO_HOST=mydomain.private" -e "RESOLVER_IP=8.8.8.8" -v /var/run/docker.sock:/tmp/docker.sock -p 80:80 hirokiky/nginx-subdomain-proxy
```

## Thanks

Originally this repository is copied from [nginx-proxy](https://github.com/jwilder/nginx-proxy).
