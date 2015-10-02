## What's new in the latest release ##

This page highlights some new features we implemented in skipfish 2.10b. The complete list of the changes can be found in the changelog:
https://code.google.com/p/skipfish/source/browse/trunk/ChangeLog


#### Configuration file support ####
As of 2.10b you can start using configuration files! Say bye bye to the long command-line and instead, simply create a copy the [config/example.cfg](https://code.google.com/p/skipfish/source/browse/trunk/config/example.conf) file and modify it for your needs.

Running a scan with a config file looks like this:

```
$ ./skipfish --config path/to/config.cfg [..other options..] http://example.org
```

The example above shows that the configuration file can still be combined with command-line flags. When doing so, please keep in mind that configuration entries are treated as long flags. This means that, for example, a flag that is only allowed to be set once (e.g. -o to specify the output directory) should not be set both in the config and on the command-line.

Interested? Check out the [example.cfg](https://code.google.com/p/skipfish/source/browse/trunk/config/example.conf) file to see all configurable options.

#### Disk flushing ####
The memory consumption can get quite high when scanning large web sites  even when using -e to discard binary responses. You can now use the --flush-to-disk flag in order to have skipfish flush server response data to disk. This will keep the memory consumption of every scan very low (e.g. 20MB for a scan that consumed 300MB).

An example command-line:
```
$ ./skipfish --flush-to-disk [..other options..] http://example.org
```

Note that the total amount of pages crawled in combination with the amount of detected issues will always influence memory consumption.

#### Updated LFI / RFI tests ####

The local file inclusion tests now use different types of encoding (e.g. double encoding).

Also in many cases, tests for web.xml failed due to Java applications requiring a payload that is an exact match with the path distance between the vulnerable page and web.xml. To overcome this we start now incrementally increase the traversal depth which gives a much higher chance to actually access the file.

Last but not least, the remote file inclusion is no longer a compile time option and instead enabled by default. If you don't like this option, you can use --checks-toggle to turn it off (or do the same in your config file). Additionally do note that the RFI URL and test string can be changed in [config.h](https://code.google.com/p/skipfish/source/browse/trunk/src/config.h#164).