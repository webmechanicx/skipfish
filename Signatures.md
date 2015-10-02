

# Introduction #

Skipfish content signatures use a Snort-like syntax and allow us to detect web application vulnerabilities, information leaks or even interesting web applications, such as phpmyadmin or phpinfo() pages.

In older versions of skipfish, content signatures were hardcoded, which made them more difficult to maintain. As of version 2.08b, we now  store signatures in external (signature) files so that they can be changed / updated or extended without having to touch any of the code.

(examples at the end of the page)

# Contributing #

The current signature list is nice but...  not complete ;-)

If you have a nice new signature or if you are able to optimize an existing one: please help out by reporting this via our issue tracker!

https://code.google.com/p/skipfish/issues/entry?template=Content%20signatures

# Signature file format #

Empty lines or lines starting with a # are skipped by the parser.  All other lines are expected to be signatures.

Signature files can use the "include" directive to include other files. This makes it easy to group signatures in different files and include ones you are interested in.

A few simple examples:

```

# Include another signature file
include signatures/mime.sigs

# A signature to detect phpinfo() pages
id:11001; sev:3; content:"<title>phpinfo()</title><meta name="; depth:2048; \
                 memo:"phpinfo() page";

# A signature to a PHP source response
id:41003; sev:4; mime:"application/x-httpd-php-source"; \
                 memo:"PHP source file (mime)";

# A signature to detect a PHP error
id:22008; prob:40402; content:"<b>Fatal error</b>: "; \
                      content:"</b> on line <b>"; depth:512; \
                      memo:"PHP error (HTML)";

```

A lot more examples can be found in the signature files we ship:
http://code.google.com/p/skipfish/source/browse/trunk/signatures/apps.sigs

# Signature keywords #

## content ##

Usage:  content:`[!]"<string>"`

The content keyword is used to specify a string that we try to match against the server response. The value can either be a static string or a regular expression (the latter requires the type:regex; modifier).

Multiple content strings can be specified per signature and, unless the signature specifies a mime type, there should be at least one.

Modifiers can be specified per content keyword to influence how the string is matches against the payload. For example, with the 'depth' keyword you can specify how far in the payload we should look for the string.

When ! is specified before the content string, the test is positive when the string is NOT present. This is mainly useful in case your signature has multiple content values.

Note: content string modifiers should be specified _after_ the content string to which they apply.

### content modifier: depth ###

Usage: content:"`<string>`"; depth:`<int>`

With depth you can limit the amount of bytes we should search for the string. Initially the depth is relative to the beginning of the payload. However, when multiple 'content' strings are used, the depth is relative to the first byte after the previous content match.

Using the depth keyword has two advantages: increase performance and increase signature accuracy.

  * Performance: A signature that matches on a `<title>` tag doesn't need to be applied to the whole payload. Instead, a depth of 512 or even 1024 bytes will help to improve performance.

  * Accuracy: In a signature with two 'content' keywords, you can force the second keyword to be searched within a very short depth of the previous content match.

### content modifier: offset ###

Usage: content:"`<string>`"; offset:`<int>`

The content string searching will start at the given offset value. For the first content string this is relative to the beginning of the payload. For the following content strings, this is relative to the first byte of the last match.

### content modifier: type ###

Usage: content:`"<string>"`; `["regex|static"]`

Indicates whether the content string should be treated as a regular expression or a static string. Content strings are treated as static by default so you can leave this keyword out unless you're using a regular expression.

In a signature that has multiple content strings, static strings can be mixed with regular expressions. You'll likely get the best performance by starting with a static string before applying a regular expression.

### content modifier: nocase ###

Usage: content:`"<string>"`; nocase;

When "nocase" is specified, the content string is matched without case
sensitivity.

This keyword requires no value.

## header ##

Usage header:`"<header name>"`

By default signature matching is performed on the respose body. By specifying a header name using the "header" keyword, this behavior is changed: the matching will occur on the header value.

The header name is not case sensitive and header signatures are treated exactly the same as content signatures meaning that you can use multiple content strings and their modifiers.

## mime ##

Usage mime:`"<string>"`

The given value will be compared with the MIME type specified by the server. This is a "begins with" comparison so a partial MIME string, like "javascript/" will match with a server value of "javascript/foo".

## memo ##

Usage memo:`"<string>"`

The memo message is displayed in the report when the signature matches. The content should be a short but meaningful problem title.

## sev ##

Usage sev:`[1-4]`

The severity with which a signature match should be reported where:

  * 1 is High
  * 2 is Medium
  * 3 is Low
  * 4 is Info (default)

## prob ##

Usage prob:`"<string>"`

All issue types are defined in database.h and, by default, signature matches are reported with generic (signature) issue types.

Using the prob keyword, a signature match can be reported as any other known issue. For example, [issue 40401](https://code.google.com/p/skipfish/issues/detail?id=40401) stands for interesting files and is already used for several signatures.

The advantage of using an existing issue ID is that it's severity and description will be used to report the signature match.

(Note this also means that the sev keyword becomes obsolete when used in combination with the prob keyword)

## check ##

Usage: check:`<int>`

Injection tests have their own ID which are specified in checks.h. Using the "check" keyword, it is possible to bind a signature to a specific injection test.

The idea is to allow context specific signatures to be written. Take the following scenario as an example: During a scan, file disclosure tests might not fully succeed to highlight a vulnerability. Errors thrown during these tests can still reveal that there is more than likely a file disclosure problem. While generic server error detection will highlight these errors, it is more useful if we can detect that these errors are related to our tests and report them as such.

## id ##

Usage id:`<int>`

The unique signature ID. Currently this is for documentation purpose only but in the future we'll probably add signature chaining which requires unique ID's as well.

## depend ##

Usage depend:`<int>`

A signature can be made dependent on another signature by specifying it's signature ID as the value of this keyword. This means that the signature will be skipped unless the dependent signature was successfully matched already.

One example use case could be a global signature that identifies a framework, say Wordpress, and dependent signatures that detect WordPress specific issues.

## proto ##

Usage proto:`"[http|https]"`

The "proto" keyword can be used to make a signature only applicable for either "http" or "https" type URLs.
This changes the default behavior where every signature is applied to both http and https URLs.

## report ##

Usage report:`"[once|always]"`

Some signatures are to find host specific problems and only need to be reported once. This can be achieved by using report:"once";

This keywords default value is "always".