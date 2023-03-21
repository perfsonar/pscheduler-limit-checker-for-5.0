# pScheduler Limit Configuration Checker for 5.0

This repository contains a program that can be used to check
pScheduler limit configurations on perfSOANR 4.0 systems for potential
problems when used with pScheduledr on perfSONAR 5.0.


## Installation

**NOTE:** This program requires libraries provided by pScheduler and
  must be run on an already-installed perfSONAR system.  It is
  recommended that the system is upgraded to the latest production
  release (4.4.6).

 * Log into an existing perfSONAR 4.x system.
 * `mkdir -p /tmp/check`
 * `cd /tmp/check`
 * `curl -L https://github.com/perfsonar/pscheduler-limit-checker-for-5.0/tarball/main | tar xz`
 * `cd perfsonar-pscheduler-limit-checker-for-5.0-*`


## Checking The Limit Configuration

To check the default limit configuration, run `./check-limits | less`

To check a different file, run `./check-limits /path/to/file | less`


## Using the Example Configuration

An example limit configuration is provided to show what sorts of
problems this program will point out.

```
$ ./check-limits ./example.conf
```

This is the output:

```
NOTE: This limit configration could not be fully validated on a perfSONAR 5.0.0
system.  This program will still attempt to find problems related to the removal
of the 'test' limit.

ACTION REQUIRED.

This limit system configuration contains limits that are deprecated in
the 4.x family and will not be supported in release 5.0.

Limit 'idleex-default':

This limit has been removed in 5.0.  The idleex test is now disallowed for
untrusted hosts by the 'allowed-tests' limit.


Limit 'some-limit-i-wrote':

This limit appears to be a customization and needs to be rewritten using the
'jq' limit.  Documentation on how to do that may be found at
https://docs.perfsonar.net/config_pscheduler_limits.html#jq-use-a-jq-script-to-make-a-pass-fail-decision.


Limit 'throughput-default-time':

This limit has been rewritten to be compatible with perfSONAR 5.0.  If yours has
not been customized, it can be replaced in its entirety with the following:
    
    {
        "name": "throughput-default-time",
        "description": "Throughput tests of reasonable duration",
        "type": "jq",
        "data": {
            "script": [
                "import \"pscheduler/iso8601\" as iso;",
                "if .test.type != \"throughput\"",
                "then true  # Don't care.",
                "else",
                "  if iso::duration_as_seconds(.test.spec.duration) > 60",
                "  then \"Duration for throughput must be 60 seconds or less.\"",
                "  else true end",
                "end"
            ]
        }
    }
    


Limit 'throughput-default-udp':

This limit has been rewritten to be compatible with perfSONAR 5.0.  If yours has
not been customized, it can be replaced in its entirety with the following:
    
    {
        "name": "throughput-default-udp",
        "description": "UDP throughput tests of reasonable bandwidth",
        "type": "jq",
        "data": {
            "script": [
                "import \"pscheduler/iso8601\" as iso;",
                "if .test.type != \"throughput\"",
                "then true  # Don't care.",
                "else",
                "   if .test.spec.udp == true",
                "     and (.test.spec.bandwidth == null",
                "          or.test.spec.bandwidth > 50000000)",
                "   then",
                "     \"UDP throughput bandwidth must be less than 50 Mb/s\"",
                "   else true end",
                "end"
            ]
        }
    }
    


Note that any of the modifications described above will be fully-compatible with
4.4.x  Getting a revised version tested and into production prior to upgrading
to 5.0 is strongly-recommended.
```
