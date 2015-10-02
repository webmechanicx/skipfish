This page describes 3 different methods you can use to run authenticated skipfish scans.





# Form authentication #

With form authentication, skipfish will submit credentials using the given login form. The server is expected to reply with authenticated cookies which will than be used during the rest of the scan.

An important feature of the form authentication is that skipfish will re-authenticate in the event the server terminates our session.

An example command-line to login using this feature:
```
$ ./skipfish --auth-form http://example.org/login \
             --auth-user myuser \
             --auth-pass mypass \
             --auth-verify-url http://example.org/profile \
             [...other options...]
```

An example configuration file / snippet to accomplish the same:

```

$ cat mylogin.conf 
# Specify the location of the login form
auth-form = https://example.com/login

# Specify the username and password.
auth-user = skipfish@gmail.com
auth-pass = skipfish

# The URL to verify authentication
auth-verify-url = https://example.com/profile

$ ./skipfish --config mylogin.conf -o output_dir http://example.com
```

## How it works ##

  * Upon start of the scan, the authentication form at /login will be fetched by skipfish. We will try to complete the username and password fields and submit the form.

  * Once a server response is obtained, skipfish will fetch the verification URL twice: once with the new session cookies and once without any cookies. Both responses are expected to be different.

  * During the scan, the verification URL will be used many times to test whether we are authenticated. If at some point our session has been terminated server-side, skipfish will re-authenticate using the --auth-form (/login in our example) .

Verifying whether the session is still active requires an URL where an authenticated request is going to get a different response than an anonymous request. Typical good URLs for this are 'profile' and 'my account' pages.

## How to troubleshoot ##

### Login field names not recognized ###

If the username and password form fields are not recognized, skipfish will complain. In this case, you should specify the field names using the --auth-user-field and --auth-pass-field flags.

### The form is not submitted to the right location ###

If the login form doesn't specify an action="" location, skipfish will submit the form's content to the form URL. This will fail in some occasions. For example, when the login page uses Javascript to submit the form to a different location.

Use the --auth-form-target flag to specify the URL where you want skipfish to submit the form to.

### Skipfish keeps getting logged out ###

Make sure you blacklist any URLs that will log you out. For example, using the " -X /logout"

### It just doesn't work ###

Run skipfish is -uv and review the results which can give hints at why it doesn't work (e.g. when you have to manually specify the form fields).  Feel free to file a bug if you're convinced things can be improved ! Else, fall back on cookie authentication with the -C flag


# Cookie authentication #

Alternatively, if the site relies on HTTP cookies you can also feed these to skipfish manually. To do this log in using your browser or using a simple curl script, and then provide skipfish with a session cookie:

```
$ ./skipfish -C name=val [...other options...]
```


Other session cookies may be passed the same way, one per each -C option.

The -N option, which causes new cookies to be rejected by skipfish, is almost always a good choice when running cookie authenticated scans (e.g. to avoid your precious cookies from being overwritten).

```
$ ./skipfish -N -C name=val [...other options...]
```

# Basic HTTP authentication #

For simple HTTP credentials, you can use the -A option to pass the credentials.

```
$ ./skipfish -A user:pass [...other options...]
```