# Common problems with skipfish (and how to fix them) #

**Note: [click here](http://code.google.com/p/skipfish/wiki/SkipfishDoc) for general project documentation. This document covers some frequent problems with _skipfish_ only.**

## Problem #1: Compilation fails. ##

The most common reason for problems here is that you do not have a properly configured build environment. Please make sure that you have the following components - and their dependencies - installed:

  * [GNU C Compiler](http://gcc.gnu.org/)
  * [GNU Make](http://www.gnu.org/software/make/)
  * [GNU C Library](http://www.gnu.org/software/libc/) (including development headers)
  * [zlib](http://www.zlib.net/) (including development headers)
  * [OpenSSL](http://www.openssl.org/) (including development headers)
  * [libidn](http://www.gnu.org/software/libidn/) (including development headers)
  * [libpcre](http://www.pcre.org/) (including development headers)

All the development headers must be in your default include path; several vendors mess this up, in which case, you might need to locate the file using `find`, and provide additional `-I/path/to/inclue/files/` options in `CFLAGS`, and `-L/path/to/libraries/` in `LDFLAGS`.

## Problem #2: My scan is taking forever. ##

The first thing to check is the number of requests per second displayed by the scanner. It should be at least 200 req/s when scanning remote targets on the Internet, and at least 1000 req/s on local networks. If it is significantly lower, your problem might be just a very slow server; in this case, try changing the `-m` parameter to see if it helps (lowering it to 2-5 may help when scanning nearby, low-latency targets; raising it to 20-30 can help with slow or non-keepalive destinations). You can also modify dictionary settings as outlined in `dictionaries/README-FIRST` to reduce the number of requests sent, at the expense of brute-force testing coverage (abridged version of this advice can be [found here](http://lcamtuf.blogspot.com/2010/11/understanding-and-using-skipfish.html)).

**Note that switching from `complete.wl` to `minimal.wl` may reduce scan time by a factor of 3; using `-Y` may result in a further 20x boost; and disabling brute-force altogether with `-L -W- ` will speed things up by a factor of 500.**

If the reported scan speed and brute-force coverage is adequate, but the scan of a relatively small site is still taking hours - you can hit `space` to switch to a preview of the current, in-flight requests to see which location is currently being scanned. If the location does not make any sense, abort the scan with `Ctrl-C` and examine the generated report. If you notice recursive, bogus nodes in the output, or other significant abnormalities, <u>please contact me right away to have it fixed</u>. In the meantime, you can use `-c` / `-x`, `-r` and `-d` options to set crawl limits, or `-X` to exclude problematic locations.

Lastly, if everything looks OK, and the site is simply very large, consider using the same `-X` and `-I` options to exclude "tarpit" locations with static content that are not worth brute-forcing. The scanner can't tell interesting places from boring ones, but you almost certainly can make this call. Quite notably, `/icons/`, `/doc/`, `/examples/`, etc, are not excluded by default - but in many cases, can be.

Keep in mind that unlike most other scanners, _skipfish_ does not try to cut corners without telling you. It does a full dictionary brute-force against every suitable location, unless explicitly instructed otherwise. This also makes it hard to display a reliable ETA: the scanner will sometimes go up to 95% only to drop to 40% when a new cluster of directories is discovered. C'est la vie.

## Problem #3: My scan is progressing too fast! ##

If you fear overloading the server, you can take throttle the amount of requests per second using the -l option. For example, using "-l 10" will force skipfish to not send more than 10 requests per second.

In addition, you can lower the number of simultaneous connections using the `-m` parameter.

Now in the rare case where -l and -m are not sufficient to slow down the scan, you can consider to use [trickle](http://monkey.org/~marius/trickle/) to limit the bandwidth available to _skipfish_.

## Problem #4: I tried it against a "demo" vulnerable application, and... ##

Simulated vulnerable applications are usually very simplistic; consider [this example](http://zero.webappsecurity.com/rootlogin.asp.bak) from SPI Dynamics: the script responds to certain predefined path inclusion attempts, e.g. `/etc/passwd`, but not to other, more versatile probes used in our code.

Some journalists and other non-technical users tend to use these vendor-supplied test sites as a benchmark for tested products; therefore, commercial scanners generally try to account for how these demo sites are designed to score well, even if there is no actual security benefit of doing so.

Given that _skipfish_ is an open-source project with no specific commercial incentive, I opted to focus on real-world coverage instead.

## Problem #5: I'm having some display issues. ##

If your terminal is set to use a non-standard palette, `skipfish` output may be harder to read. If this is a problem, edit `config.h` and comment out the `USE_COLORS` line, then recompile.

Another problem is that if your terminal is set to 80x25, `skipfish` runtime statistics may not fit the screen neatly. If this is the case, please resize to at least 100x35.

If you are simply annoyed by the verbose runtime reporting, or are using a slow terminal, you can also use `-u` to suppress all the real-time info.

## Problem #6: I can't view the report in Safari or Chrome. ##

This is likely because you browsed to the report via the `file:///` protocol. Certain important security improvements implemented in these two browsers have the unfortunate side effect of crippling local Java<b></b>Script.

In Chrome, you can work around this by passing a command-line option of `--allow-file-access-from-files`. You can also put the report in a local WWW root and navigate to `http://localhost/...` instead; or use Firefox.

## Problem #7: Scan output directory is huge, what is going on? ##

In addition to the list of issues found, the directory contains all the noteworthy HTTP requests and responses seen (including copies of all discovered files), simply to confirm and document scanner findings. While this may result in a rather sizable blob of data, it does not affect the report-viewing experience in any way.

If you do not want to keep this evidence, you can use `-e` to suppress storing and saving the contents of binary files; or simply delete all the `*.dat` files recursively.

## Problem #8: Memory footprint is huge, what is going on? ##

Skipfish should not leak any memory, and uses relatively little of it to represent internal data - but it does keep in-memory cache of all crawled documents. The default sample size limit is rather generous: 200 kB. Therefore, if your site consists of 30,000 large videos, you can expect several gigs to be consumed.

There are several simple ways to fix this: use `-e` to stop the scanner for storing binary files for reporting purposes; specify `-c` or `-x` to limit the number of nodes indexed; `-I` to narrow down the scan to interesting, dynamic locations on your server; use `-X` to exclude paths with static content tarpits (or skip certain extensions); or provide `-s` to limit sample size.

Ultimately, it would probably make sense to store samples for which all analysis is completed on the disk, rather than in memory; patches welcome.

## Problem #9: The scanner is causing tons of 404 errors! ##

Well, it's supposed to - an important part of its functionality is brute-force to discover hidden, accidentally exposed resources on the server; a vast majority of these probes will  yield 404s.

That said, you can use the tool for an orderly crawl with no brute-forcing steps; see `dictionaries/README-FIRST` for advice on how to create and use a null dictionary.

## Problem #10: I am seeing very random false positives and non-existent directories. ##

You might have stumbled upon a real bug in _skipfish_ - in this case, see the next section for info on how to report it. That said, one of the common causes of very erratic scans are web frameworks that start to behave inconsistently under load due to race conditions, backend database limits, and so forth. Since this, by itself, may be a security problem - it makes sense to investigate.

Consider this real-world example of a request pattern that led _skipfish_ to assume a shell injection vulnerability:

```
* URL http://127.0.0.1/manual/custom-error.html/'`false`' (200, len 9919)
* URL http://127.0.0.1/manual/custom-error.html/'`true`' (200, len 9919)
* URL http://127.0.0.1/manual/custom-error.html/'`uname`' (200, len 8688)
```

The two first responses were identical, and the last one differed significantly. The differential probe did what it was supposed to: presence of an underlying vulnerability was the only logical conclusion to make under normal circumstances. In reality, the server simply acted up, returning an incomplete page on that last request.

Another example of a similar pattern that caused _skipfish_ to randomly recurse into non-existent directories:

```
* URL http://127.0.0.1/manual/custom-error.html/en/ (200, len 3673)
* URL http://127.0.0.1/manual/custom-error.html/en/sfi9876 (200, len 7769)
```

When retried later on, both requests return exactly the same data.

To combat this problem, _skipfish_ does some initial server behavior checks - by default, sending 15 consecutive vanilla requests to every fuzzed location, and bailing out early if any of the responses is significantly different than the rest. This short test is not always enough to catch more elusive cases: if you want to make the check more sensitive, you can edit `config.h` and raise the `BH_CHECKS` limit to send more probes (at the expense of prolonging the scan).

The best way to track down the issue is to examine the scan log and request traces, examine problematic responses to see why they differ, and investigate server logs. If you have a difficulty figuring it out, some manual stress-testing may help. Lowering the `-m` parameter may reduce the incidence of such glitches, too; `-Z` option will prevent the crawler from ever descending into HTTP 500 directories, which may help, too.

In some rare cases, the investigation of these false positives may lead you to non-synchronized load-balancing backends, or features that insert random quotes on a page for amusement purposes. My strong recommendation is to correct the underlying problem or disable the QOTD functionality before continuing; if this is not possible, adjusting the `FP_*` values in `config.h` to make the page comparator less pedantic may also help, but will have side effects.

## Problem #11: I am really annoyed by the splash screen. ##

Sorry; this proved to be necessary - quite a few people were misconfiguring skipfish, and then complaining about the duration of a scan, or the number of requests made.

You can edit `config.h` and comment out the `SHOW_SPLASH` line, then recompile, to permanently disable this notice.

## Problem #12: I don't understand the point of all these MIME- or charset-related warnings. ##

Missing or mismatched MIME types on any files with user-controlled contents may easily lead to cross-site scripting flaws. This is because of [browser content sniffing logic](http://code.google.com/p/browsersec/wiki/Part2#Survey_of_content_sniffing_behaviors), a problem is particularly pronounced in Microsoft Internet Explorer - where even something as subtle as returning `image/jpeg` on a GIF file may cause HTML detection to be attempted, possibly interpreting any HTML stuffed in the EXIF fields of an otherwise valid image.

The same applies to the absence of valid and unambiguous charset directives; any mistakes may prompt [automatic character set detection](http://code.google.com/p/browsersec/wiki/Part2#Character_set_handling_and_detection) in the browser, and some of the character sets that may be detected based on user input will profoundly change the meaning of the document; [UTF-7](http://en.wikipedia.org/wiki/UTF-7) is a striking example of this. Even something as subtle as mistyping `utf-8` as `utf8` may lead to trouble.

## Problem #13: You need to add JS support! ##

As should be apparent, _skipfish_ has no elaborate support for Java<b></b>Script - which also means it gets only an average score in <a href='http://code.google.com/p/wivet/'>wivet</a> and similar JS-heavy link extraction tests. That said, adding a script-executing engine to a crawler is a significant undertaking - and I am not entirely convinced that it's actually worth the effort.

The fundamental problem is that while it enables more sophisticated link extraction for simple `onload` or `onclick` handlers, it still leaves the tool completely clueless as to how to meaningfully interact with the increasingly complex client-side logic we are seeing in the wild. Think of auditing an application such as Google Docs or Google Mail; in extreme cases, the server is only a dumb storage mechanism with basic access control - and everything else is done in Java<b></b>Script.

Past a certain point, JS-rich pplications may be simply impossible to usefully test in a fully automated manner; to help there, <a href='http://code.google.com/p/ratproxy/'>ratproxy</a> might be a better choice.

## For issues not listed here... ##

For any problems not covered on this page, and **particularly** for:

  * Non-existent files mysteriously appearing in the report,
  * Scans that recursively descent into non-existing paths (`/foo1/foo2/foo1/foo2/...`),
  * Vulnerabilities that do not seem to be present in reality (false positives),
  * Vulnerabilities missed by the scanner (false negatives),
  * New vulnerabilities or interesting files it should be checking for,
  * Cases where MIME types are incorrectly identified on a page,
  * Crashes during a scan.

...**please** contact us per email ([lcamtuf@google.com](mailto:lcamtuf@google.com) and [heinenn@google.com](mailto:heinenn@google.com)) . Complaining on Twitter will not get the bug fixed :-)

Before filing a report, you may also want to have a look at the general [issue reporting tips](http://code.google.com/p/skipfish/wiki/SkipfishDoc#Oy!_Something_went_horribly_wrong!) to learn how to collect log data or meaningful crash dumps.