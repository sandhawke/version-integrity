# The `"version-integrity="` URL convention

This page documents a convention for adding security to immutable web
resources. The security comes from clients confirming secure hashes of
the content.  It can be done without any server-side changes, although
servers can help and get some benefits. This is similar to
[Subresource Integrity](https://www.w3.org/TR/SRI/), but works without
changing nearly as much of stack. It may be especially useful for
linked data applications.

The convention is that if a URL contains the substring
"version-integrity=", then the URL should be understood as
identifying an immutable resource.  Clients can detect that substring
and then (1) assume the resource is immutable, so it can be cached forever,
and (2) compute a secure hash of the resource representation and make
sure it matches the hash embedded in the URL.

For example, <https://www.w3.org/People/Sandro/ping.txt> is currently
a small 5-byte resource, with the text "pong\n".  I could change that
text, or someone else could.  It's not really immutable.

Meanwhile,
<https://www.w3.org/People/Sandro/ping.version-integrity=sha256-Wmoo_BYA6hQdezkSWCLB1R-xZqvlYo5_wfmamwL11Sw=.txt>
has the same 5 bytes.  But this URL includes a secure hash of those 5
bytes.  Clients can do the fetch and check the hash.  If it matches,
they can proceed normally.  If the hash doesn't match, then they know
something is wrong, and it's probably not safe to proceed.

Following this convention, if I give someone that URL, they know I
intend it to be immutable, and they can and should check it, before
relying on the contents.  In this case, the version-integrity
substring is a part of the filename, on this static-content part of
the server.  That was easy to do, and shows the resource owner's intent.

As an alternative, without support from the server or the resource owner, people can
compose fragment URLs, like:,
<https://www.w3.org/People/Sandro/ping.txt#version-integrity=sha256-Wmoo_BYA6hQdezkSWCLB1R-xZqvlYo5_wfmamwL11Sw=>.
Clients can check this the same way, but the fragment part (after the
#) isn't sent to the server. The fragment semantics might be an issue in some applications.

Another alternative is to use a query URL, like <https://www.w3.org/People/Sandro/ping.txt?version-integrity=sha256-Wmoo_BYA6hQdezkSWCLB1R-xZqvlYo5_wfmamwL11Sw=>.  Most servers (like www.w3.org) will ignore unexpected query parameters, so this works fine.  This also gives servers a chance to offer up a specific immutable version of a mutable resource, if they want.

From the client perspective, these are all the same.  The client
simply notices the "version-integrity=" substring of the URL and makes
sure to check whatever content ends up being returned.

## Computing the hash

We'd like to use base64 encoding like [Subresource
Integrity](https://www.w3.org/TR/SRI/) but having slash as one of the
characters get really messy, so we use base64url, which is trivially
different.  Here are two different ways to compute the hash with
common, existing command-line tools:

```
$ sha256sum < ping.txt | tr -d - | hex -d | base64url
Wmoo_BYA6hQdezkSWCLB1R-xZqvlYo5_wfmamwL11Sw=

$ openssl dgst -sha256 -binary ping.txt | openssl base64 -A | tr '+/' '-_'
Wmoo_BYA6hQdezkSWCLB1R-xZqvlYo5_wfmamwL11Sw=
```

## Implementations

I've made a node.js implementation that works as a library or from the command line: [got-integrity](https://npmjs.org/package/got-integrity)

If you have a recent node.js installed, you can try it without any other install like this:

```
$ npx got-integrity --check https://www.w3.org/People/Sandro/ping
```

Please let me know about other implementations.

## Details

(more details as they arise)

## Issues

See [issue tracker](https://github.com/sandhawke/version-integrity/issues)

