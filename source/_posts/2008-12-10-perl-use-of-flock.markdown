---
layout: post
title: "Perl: Use of flock"
date: 2008-12-10 20:45
comments: true
categories: perl script example
---
This is a small example on flock. It may help you to prevent multiple running instances of the same script. Assume you run the script via cron and it may not be finished when cron attempts to start it again. This few lines of code solve this issue.

{% codeblock flock example lang:perl %}
if ( ! open( LOCK, ">/var/run/my_app.lock" ) ) {
    print "Failed to open lock file: $!\n";
    exit(1);
}
 
## Create exclusive, non blocking lock: LOCK_EX(2) + LOCK_NB(4)
if ( ! flock( LOCK, 6 ) ) {
    close( LOCK );
 
    if ( ! open( PID, "</var/run/my_app.pid" ) ) {
 
        print "Failed to read PID file: $!\n";
        exit(1);
    }
 
    my $pid = <PID>;
    close( PID );
 
    print "Failed to accquire lock. Another instance (PID $pid) running!\n";
    exit(1);
}
else {
    if ( ! open( PID, ">/var/run/my_app.pid" ) ) {
 
        print "Failed to open pid file: $!\n";
        exit(1);
    }
 
    print PID $$;
 
    if ( ! close( PID ) ) {
        print "Failed to write PID file: $!\n";
        exit(1);
    }
}
{% endcodeblock %}

