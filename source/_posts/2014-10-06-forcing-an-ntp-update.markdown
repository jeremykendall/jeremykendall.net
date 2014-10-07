---
layout: post
title: "Forcing an NTP Update"
date: 2014-10-06 09:44:01 -0500
comments: true
categories: ["devops", "virtual machines", "ntp"]
---

**UPDATE**: Dan Horrigan [pointed out a much better solution](https://twitter.com/dhrrgn/status/519166134537310208) 
to this problem on Twitter, one that has nothing to do with NTP at all. He adds 
a VirtualBox config option in his `Vagrantfile`s to update the virtual
machine's time from the host every 10 seconds. Nice!

{% gist 9e31f6de41d83f21107a %}

**UPDATE 2**: Oops. I didn't explain that correctly. Here's Dan to set the matter straight:

<blockquote class="twitter-tweet" lang="en"><p><a href="https://twitter.com/JeremyKendall">@JeremyKendall</a> that is slightly wrong. It sets the drift threshold, so it only syncs if it is 10 seconds or more off.</p>&mdash; Dan Horrigan (@dhrrgn) <a href="https://twitter.com/dhrrgn/status/519246842769727488">October 6, 2014</a></blockquote>
<script async src="//platform.twitter.com/widgets.js" charset="utf-8"></script>

**ORIGINAL POST**: I frequently find myself needing to update the time on my Vagrant boxes.
This is especially true when I'm testing [Query Auth][1] between two different
Vagrant boxes (mimicking a client/server relationship) as Query Auth 
will fail a request with a timestamp that varies too greatly from the server's 
timestamp (+-15 seconds by default). Even though I run [NTP][2] on all my virtual
servers, time drift is a frequent problem. Furthermore, when the drift is too great 
(can't find a reference to how large the drift needs to be), NTP won't correct
it all at once.

A quick Google search later and, once again, it was Stack Overflow the rescue, 
courtesy of [Martin Schröder's answer][3] to the question ["How to force a clock update using ntp?"][4]

One quick bash script later and excessive clock drift is now a trivial issue.

```
#!/bin/bash
# Forces an ntp update
#
# Based on SO user Martin Schröder's answer to "How to force a clock update
# using ntp?": http://askubuntu.com/a/256004/41943

# Fail fast (set -e will bail at first error)
set -e

if [ "$EUID" -ne 0 ]; then
    echo "ERROR: '$0' must be as root."
    exit 1
fi

service ntp stop

echo "Running 'ntpd -gq'"
ntpd -gq

service ntp start
```

## Postscript

If you're unfamiliar with the particulars of creating bash scripts, here are the
steps you can take to create your own version of the above script. You may need 
to preface the commands using `sudo`.

* Copy and paste the above script into a text file (I've named mine `force-ntp-update`).
* On the command line, call `chmod a+x /path/to/force-ntp-update` to allow all users to execute the command.
    * If you're the only user who should be able to execute the command, change the above to `chmod u+x /path/to/force-ntp-update`.
* Move the script somewhere in your path: `mv /path/to/force-ntp-update /usr/local/bin`.
* Done!

Now anytime you need to force an update, simply call `sudo force-ntp-update`.

[1]: https://github.com/jeremykendall/query-auth
[2]: http://www.ntp.org/
[3]: http://askubuntu.com/a/256004/41943
[4]: http://askubuntu.com/questions/254826/how-to-force-a-clock-update-using-ntp
