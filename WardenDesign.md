# Introduction #

Warden is a system for securing paths to files and folders based on a simple shared secret encrypted token which is created when the url to the content is created and validated when the content is requested from the server. Security comes through the ability to set an expiration time for the token and thus the content. Anyone sharing that token can only access the content for the life of the token. By default, this is not a system for one time downloads, but instead a system to mitigate people sharing links to your content. Warden can be expanded to support one time downloads, but that requires registering each token with the server and is beyond the scope of this initial document. The beauty of Warden is that it is simple and also works with content delivery networks (CDN).

# Example #

When dynamically generating HTML, you make links to content that needs to be secured. This content can be images, files, or whatever you can imagine. At the end of the link, you append a query string parameter that ends up looking like this:

```
http://foo.com/path/to/my/secure/file.jpg?t=234232234%7Clkjalkj23lk4j23lk4jl23j4lk23j4l
```

Breaking things down a bit, the token is represented by the letter t. The value of the token is:

```
a=US,CA
d=FR
allowedPaths = regex based queryString formatted paths
encryptedText = encrypt(expirationTimestamp|a=allowedLocation&d=disallowedLocation|customQueryStringData|allowedPaths)
t=urlEncode(expirationTimestamp|encryptedText)
```

So, for the above example, the data would look like this:

```
encryptedText = encrypt(234232234|||/path/to/my/secure/file.jpg)
t=urlEncode(234232234|encryptedText)
```

That URL and its query parameters is passed to a server which then decrypts the token, runs any Warden plugins passing in the allowedLocation/disallowedLocation (if they exist) and customQueryStringData and then validates the allowedPaths. If there is invalid content, Warden will return HTTP\_UNAUTHORIZED.

Warden has an additional feature in that you can use a cookie to secure the entire contents of a page with only one token. For example, if you have a page with multiple images on it that need to be secure, then you can generate a token that looks like this:

```
encryptedText = encrypt(234232234|||/path/to/my/secure/*.jpg)
```

Assuming all your content resides on a server for which requests with the cookie will be sent, then you just place that token in a cookie called wardenToken. In other words, if you generate the cookie on .foo.com, then your content needs to live on either .foo.com or something like content.foo.com.