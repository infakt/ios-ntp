# ios-ntp

A network time protocol client (and an application testbed for iOS).
This is a continues to be a work in progress.

Created by Gavin Eadie on Oct 17, 2010

### News
**July 22, 2015:** (version 1.1.1) `ios-ntp` has contained a resource file 
(called `ntp.hosts`) which contained a list of time server hosts to be used
for querying the time.  That file has been removed in this release.

The logic is that, since `ios-ntp` can now be added to a project via CocoaPods, 
any local changes made to that file will be overwritten the next time the `ios-ntp`
pod is updated and, since `ios-ntp` already contains a built-in list of time servers,
removing this file from the pod should not impact the behavior of the `ios-ntp` code.

The `ntp.hosts` file was always meant to over-ride the default host list -- that 
intent is now better served.  The `ntp.hosts` file used to contain:

		#
		# tasteful worldwide time server name collection
		#

		0.pool.ntp.org
		#1.pool.ntp.org
		#2.pool.ntp.org
		#3.pool.ntp.org
		0.uk.pool.ntp.org
		#1.uk.pool.ntp.org
		#2.uk.pool.ntp.org
		#3.uk.pool.ntp.org
		0.US.pool.ntp.org
		#1.US.pool.ntp.org
		#2.US.pool.ntp.org
		#3.US.pool.ntp.org
		asia.pool.ntp.org
		europe.pool.ntp.org
		north-america.pool.ntp.org
		oceania.pool.ntp.org
		south-america.pool.ntp.org
		africa.pool.ntp.org
		#time.apple.com

.. which is exactly the same as the internal default list (though that has no
commented host addresses).

If you want to use your own list of time servers, you need to create a file containing
time host names, one per line, name it `ntp.hosts` and place it in the main bundle of your
application (the sample app `ios-ntp-app` does this to use the server at `time.apple.com`).

**June 10, 2015:** (version 1.1) I recently discovered a re-entrancy bug when John 
Grismore brought my attention to inaccuracies in reported network time
offsets. When
a NetAssociation notified the NetClock that it had a new time
offset, that event might interrupt the offset averaging that NetClock
does causing the averaging to break.

In fact, this mechanism isn't optimal anyway!  The notifications
that cause offset averaging arrive at NetClock constantly whether
the result is used or not.  We're keeping the NetClock network time 
offset property up
to date whether we need it or not.  Better would be to perform the
averaging only when the offset NetClock property is called for, and 
that is how
ios-ntp now works.

The API is not changed, so your application should require no changes.

**February 22, 2015:** Several important changes have been made
including one that will be helpful for those who want to get a quick
one-time value of the difference between system time and network time.

Before this change, ios-ntp would use time estimates from a set
of NetAssociations (one per time servers), constantly determining the best
time by sampling these values.  This is the model for computers which
have a continuous low level task monitoring the time.  Application in
iOS often have a different need; they are more likely to want an fast
estimate of the time on demand.  To provide the ability to the developer,
access has been provided to use a NetAssociations directly

An NetAssociation can now be asked for one measure of the time 
from one time server so an iOS app can create an NetAssociation, use 
it to get the time, and be done.

This code operates on 32-bit and 64-bit iOS devices.

This code **requires** iOS 7, or higher.

----
### About

The clock on the oldest iPhone, iTouch or iPad is not closely
synchronized to the correct time. In the case of a device which is
obtaining its time from the telephone system, there is a setting to
enable synchronizing to the phone company time, but that time has been
known to be over a minute different from the correct time.

In addition, users may change their device time and severely affect
applications that rely on correct times to enforce functionality, or may
set their devices clock into the past in an attempt to dodge an expiry
date.

This project contains code to provide time obtained from standard time
servers using the simple network time protocol (SNTP: RFC 5905). The
implementation is not a rigorous as described in that document since the
goal was to improve time accuracy to tens of milliSeconds, not to
microseconds.

Computers using the NTP protocol usually employ it in a continuous low
level task to keep track of the time on a continuous basis.  A
background application uses occasional time estimates from a set of time
servers to determine the best time by sampling these values over time.
iOS applications are different, being more likely to want a one-time,
quick estimate of the time.

ios-ntp provides both the continuous and on-demand modes of operation.
The continuous mode uses multiple 'associations' with time servers which
use timers to repeatedly obtain time estimates.  These associations can,
however, be used by the developer to get one time from one server.

### Usage

The code can be incorporated as source code or as a framework in an
Xcode project.  The framework usage is temporarily unavailable but will
be restored soon.

_More to come about using a framework._

Download the [ios-ntp](http://github.com/jbenet/ios-ntp) project, add
the necessary to your project, build and run.  You will need:

		#import "ios-ntp.h"

where ios-ntp is referenced.

##### Continuous Mode

Simply create a `NetworkClock`.  As soon as you create it, the NTP
process will begin polling the time servers in the "ntp.hosts" file (if
the file isn't found, a tasteful set of default servers will be used).
You may wish to start it when the application starts, so that the time is
well synchronized by the time you actually want to use it, just call it
in your AppDelegate's `didFinishLaunching` method.:

		NetworkClock * nc = [NetworkClock sharedNetworkClock];

then wait at least ten seconds for some time servers to respond before
calling:

		NSDate * nt = nc.networkTime;

It will take about one minute before untrustworthy servers start to get
dropped from the pool.

_It would probably be better if NetworkClock called back to a delegate
method, like NetAssociation does below, when it had a good time but
that's not how it works, yet, so you have to wait till things settle
down._

##### On Demand Mode

This usage is slightly more complicated.  The developer must create an
`NetAssociation` (with some specified time server), and then tell it get the
time from that server.  The association uses a delegate method to return
itself with time information.

		netAssociation = [[NetAssociation alloc] initWithServerName:@"time.apple.com"];
		netAssociation.delegate = self;
		[netAssociation sendTimeQuery];

		...

		- (void) reportFromDelegate {
		   double timeOffset = netAssociation.offset;
		}

### Performance

iOS is an event driven system with an emphasis
on rapid response to gestures at the cost of other activity. This
encourages the extensive use of design patterns like notification and
delegation so, I think, the calculation small time differences in this
environments suffers as a result.

Empirical observations of one time server shows some an occasional time
offset that is significantly greater than its usual values; the
calculated standard deviations of any one server's offsets is higher
than would be expected, and I don't know the cause of this.

### License

The [MIT](http://www.opensource.org/licenses/mit-license.php)
License Copyright (c) 2010-2015, Ramsay Consulting

### Building

_More to come about building a framework._

### History

**November 19, 2014:** A large update was made today to bring ios-ntp
into the modern world. The changes do include one bug fix, but are
mostly related to making the code comply with the recent Xcode changes
and requirements.

Some of jbenet's "Usage" notes below aren't completely accurate as a
result of these changes, and I will update the text soon.

Finally, note that this code was first written when there were only
32-bit iOS devices. As I write this there are still 32-bit devices which
run the latest version of iOS (iPhone 4S, for example), but all newer
iOS devices have a 64-bit architecture (iPhone 6, for example), and
Apple requires that this be supported.