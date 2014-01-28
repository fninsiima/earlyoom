earlyoom - The Early OOM Daemon
================================
The oom-killer generally has a bad reputation among Linux users. This may be
part of the reason Linux invokes it only when it has absolutely no other choice.
It will swap out the desktop environment, drop the whole page cache and empty
every buffer before it will ultimately kill a process. At least that's what I
think what it will do. I have yet to be patient enough to wait for it.

Instead of sitting in front of an unresponsive system, listening to the grinding
disk for minutes, I usually press the reset button and get back to what I was
doing quickly.

If you want to see what I mean, open something like
http://www.unrealengine.com/html5/
in a few Firefox windows. Save your work to disk beforehand, though.

The downside of the reset button is that it kills all processes, whereas it 
would probably have been enough to kill a single one. This made people wonder
if the oom-killer could be configured to step in earlier: [superuser.com][2]
[unix.stackexchange.com][3]
As it turns out, no, it can't. At least not in the kernel.

In the user space however, we can do whatever we want. 

What does it do
---------------
earlyoom monitors the amount of available memory. "Available memory" is what
free(1) calls "free -/+ buffers/cache", corrected for tmpfs memory usage
[(more info)][1]. In the output below, the available memory is 5918376 kiB.
That number is checked 10 times a second.

	free
		         total       used       free     shared    buffers     cached
	Mem:       7894520    5887116    2007404    1519648     245436    3665536
	-/+ buffers/cache:    1976144    5918376
	Swap:            0          0          0

When that number drops below 10% of your total physical RAM, it will kill -9 the
process that has the most resident memory ("VmRSS" in /proc/*/status).

Note that swap is not taken into account at all. That means that processes
may get killed before swap is used at all. This, however, is the point of
earlyoom: To keep the system responsive at all costs. earlyoom is designed
for systems with lots of RAM (6 GB or more) and with swap disabled.

Why not trigger the kernel oom killer?
--------------------------------------
Earlyoom does not use "echo f > /proc/sysrq-trigger" because the Chrome people made
their browser always be the first (innocent!) victim by setting oom_score_adj
very high ( https://code.google.com/p/chromium/issues/detail?id=333617 ).
Instead, earlyoom finds out itself by reading trough /proc/*/status
(actually /proc/*/statm, which contains the same information but is easier to
parse programmatically).

How much memory does earlyoom itself use
-----------------------------------------
About 0.6MB RSS. All memory is locked using mlockall().

Download and Compile
--------------------
Easy:

	git clone https://github.com/rfjakob/earlyoom.git
	cd earlyoom
	make
	sudo make install # Optional, if you want earlyoom to start automatically (works on Fedora)

Use
---

	./earlyoom

It will inform you how much physical RAM you have, how much is currently
available and what the minimum amount of available memory is:

	earlyoom v0.1-11-gce9a323
	kb_total:  7894480
	kb_min:     789440
	kb_avail:  5350448

If the available memory drops below the minimum, processes are killed until it
is above the minimum again.

Contribute
----------
Bug reports and pull requests are welcome via github. In particular, I am glad to
accept
* An init script that works on Debian/Ubuntu
* Use case reports and feedback

[1]: http://www.freelists.org/post/procps/library-properly-handle-memory-used-by-tmpfs
[2]: http://superuser.com/questions/406101/is-it-possible-to-make-the-oom-killer-intervent-earlier
[3]: http://unix.stackexchange.com/questions/38507/is-it-possible-to-trigger-oom-killer-on-forced-swapping
