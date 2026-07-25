# nginx 安全加固

综合参考资料：[webdock.io](https://webdock.io/en/docs/how-guides/security-guides/how-to-configure-security-headers-in-nginx-and-apache)

* [禁用服务器标识信息（server tokens）](https://nginx.org/en/docs/http/ngx_http_core_module.html#server_tokens)

  ```nginx
  server_tokens off;
  ```

* [内容安全策略参考](https://content-security-policy.com/)

  ```nginx
  add_header Content-Security-Policy "default-src 'self';" always;
  ```

* [X-Frame-Options](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/X-Frame-Options#sameorigin)

  ```nginx
  add_header X-Frame-Options SAMEORIGIN always;
  ```

* [X-XSS-Protection：阻止模式](https://docs.nginx.com/nginx-management-suite/acm/how-to/policies/proxy-response-headers/)

  ```nginx
  add_header X-Xss-Protection "1; mode=block" always;
  ```

* [严格来源策略（strict-origin）](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy#strict-origin)

  ```nginx
  add_header Referrer-Policy "strict-origin" always;
  ```

* [权限策略](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy)

  ```nginx
  add_header Permissions-Policy "geolocation=(),midi=(),sync-xhr=(),microphone=(),camera=(),magnetometer=(),gyroscope=(),fullscreen=(self),payment=()";
  ```

* [内容嗅探](https://en.wikipedia.org/wiki/Content_sniffing)

  ```nginx
  add_header X-Content-Type-Options nosniff always;
  ```
