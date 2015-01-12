---
layout: post
title: Thunderbird sucks, Claws owns!
date: '2008-07-02T01:46:00-04:00'
tags:
- Config
- Code
- mail
- thunderbird
- claws
tumblr_url: http://www.hisham.cc/post/30203537872/thunderbird-sucks-claws-owns
---
I was setting up my signature file (~/.sig) in Thunderbird today, and I thought to myself, “hmm, let me create a fifo instead, and pipe some output from fortune into it”. So I mkfifo''ed ~/.sig and wrote a little perl script to write out my signature into the fifo when Thunderbird asked for it. The script is pretty simple:

#!/usr/bin/perl -w

chdir;
$FIFO = ''.sig'';

while (1) 
{
    unless (-p $FIFO) 
    {
        unlink $FIFO;
        system(''mknod'', $FIFO, ''p'') 
            &amp;&amp; die "can''t mknod $FIFO: $!";
    }

    # next line blocks until 
    # there''s a reader
    open (FIFO, "&gt; $FIFO") 
    || die "can''t write $FIFO: $!";
    print FIFO &lt;&lt;EOF
--
HMB.
(hisham.mardambey\@gmail.com)
Codito Ergo Sum.

EOF
;
    print FIFO `fortune`, "\n";
    close FIFO;
    sleep 5;    # to avoid dup signals
}


The result of which will be:

--
HMB.
(hisham.mardambey@gmail.com)
Codito Ergo Sum.

Guy in chicken costume:  The world is gonna end at midnight tonight. Y2K. 
Peter Griffin:  Y2K? What are you selling, chicken or sex jelly?


And to my amazement, as soon as I fired up Thunderbird and tried to compose a new email, the entire user interface blocked, and my CPU usage went through the roof. A quick check showed that Thunderbird was infinitely reading from the fifo. Son of a … After some google’ing around, I found this to be a common bug in the 2.x.x series, so I upgraded to 3.x.x, and, they had introduced a “fix”. What sort of fix might you ask? Well, I could compose a message alright, except the ~/.sig file wouldn’t get read at all. What a fix! If the file is a fifo don’t read it? Thats hilarious. At this point, I was fed up, Thunderbird was going away. I remembered another mail client I used to use, Claws. A quick call to emerge installed it, and 2 minutes later, I had it all set up and it was reading my ~/.sig file properly. Claws 1, Thunderbird 0.
