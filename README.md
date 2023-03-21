# pScheduler Limit Configuration Checker for 5.0

TODO: Write this.


## Using the Example Configuration

```
$ ./check-limits ./example.conf

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

This limit appears to be a customization and needs to be rewritten using the
'jq' limit.  Documentation on how to do that may be found at
https://docs.perfsonar.net/config_pscheduler_limits.html#jq-use-a-jq-script-to-make-a-pass-fail-decision.


Note that any of the modifications described above will be fully-compatible with
this system.  Testing them prior to  upgrading to 5.0 is strongly-recommended.
```
