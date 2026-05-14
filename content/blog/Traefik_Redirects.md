+++
title = "Handling Redirects with Traefik Reverse Proxy"
date = 2025-08-25
[taxonomies]
tags = ["Linux", "Docker", "DevOps", "Homelab", "Server"]
+++
Handling redirects with Traefik involves configuring middlewares to manage different types of redirections. Traefik supports several types of redirections, including RedirectScheme and RedirectRegex, which can be configured in various formats such as YAML, TOML, and Docker labels.

For RedirectScheme, you can configure Traefik to redirect clients to a different scheme (e.g., from HTTP to HTTPS) by setting the `scheme` and `permanent` options. For example:

```yaml
http:
  middlewares:
    test-redirectscheme:
      redirectScheme:
        scheme: https
        permanent: true
```

This configuration will redirect all HTTP requests to HTTPS permanently

For RedirectRegex, you can use regular expressions to match and replace parts of the URL. For instance, to redirect requests from `http://localhost/` to `http://mydomain/`, you can configure it as follows:

```yaml
http:
  middlewares:
    test-redirectregex:
      redirectRegex:
        regex: "^http://localhost/(.*)"
        replacement: "http://mydomain/${1}"
        permanent: true
```

This configuration uses a regular expression to match the incoming request and replaces it with the specified URL

When configuring redirects, it's important to ensure that any reverse proxies between the client and Traefik are trusted to avoid issues with X-Forwarded headers

Additionally, you can use Docker labels to apply these configurations directly to a container. For example, to redirect all requests to an external domain, you can configure the labels as follows:

```yaml
labels:
  - "traefik.http.routers.redirect-router.entrypoints=web"
  - "traefik.http.routers.redirect-router.rule=Host(`redirect.localhost`)"
  - "traefik.http.routers.redirect-router.middlewares=redirect-regex"
  - "traefik.http.middlewares.redirect-regex.redirectregex.regex=(.*)"
  - "traefik.http.middlewares.redirect-regex.redirectregex.replacement=https://externaldomain.com"
  - "traefik.http.middlewares.redirect-regex.redirectregex.permanent=false"
```

This configuration will redirect all requests to `redirect.localhost` to `https://externaldomain.com`

When testing redirects, it's recommended to set the `permanent` option to `false` to avoid caching issues in the browser

For more detailed configurations and examples, you can refer to the Traefik documentation on RedirectScheme and RedirectRegex.
&ensp;

## Handling multiple regex capture groups
In the context of the `redirectRegex` middleware in Traefik, the `${1}` is a reference to a capture group in the regular expression.

In the example I provided earlier:
```yaml
http:
  middlewares:
    test-redirectregex:
      redirectRegex:
        regex: "^http://localhost/(.*)"
        replacement: "http://mydomain/${1}"
        permanent: true
```
The regular expression `^http://localhost/(.*)` contains a capture group `(.*)`, which matches any characters (including none) after the `/` in the URL. The parentheses `(` and `)` create a group, which allows you to reference the matched text later.

The `${1}` in the `replacement` field is a reference to the first capture group (i.e., the group created by the parentheses). It will be replaced with the actual text matched by the group.

For example, if the incoming request is `http://localhost/path/to/resource`, the regular expression will match the URL and capture the `path/to/resource` part in the group. The `${1}` in the replacement string will be replaced with `path/to/resource`, resulting in the final redirect URL: `http://mydomain/path/to/resource`.

In general, the syntax `${n}` refers to the nth capture group in the regular expression, where `n` is a positive integer. You can have multiple capture groups in a regular expression and reference them in the replacement string using `${1}`, `${2}`, `${3}`, and so on.

Here's another example with multiple capture groups:
```yaml
http:
  middlewares:
    test-redirectregex:
      redirectRegex:
        regex: "^http://localhost/([^/]+)/([^/]+)"
        replacement: "http://mydomain/${1}/${2}"
        permanent: true
```
In this case, the regular expression `^http://localhost/([^/]+)/([^/]+)` has two capture groups: `([^/]+)` matches one or more characters that are not `/`, and the second `([^/]+)` matches another set of characters that are not `/`. The replacement string `http://mydomain/${1}/${2}` will be replaced with the actual text matched by the two groups.

For instance, if the incoming request is `http://localhost/user/profile`, the regular expression will match the URL and capture `user` and `profile` in the two groups. The replacement string will be replaced with `http://mydomain/user/profile`.
