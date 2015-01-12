---
layout: post
title: mod_msg_filter - block/allow message delivery over HTTP for ejabberd
date: '2012-04-28T16:48:00-04:00'
tags:
- Code
- mod_msg_filter
- erlang
- ejabberd
tumblr_url: http://www.hisham.cc/post/30203546951/mod-msg-filter-block-allow-message-delivery-over
---
One of the requirements we have for our chat service where I work is
the ability to decide whether users are allowed to chat based on
business logic that we execute in our permission system. This means
that we need to ask the permissions system every time people start to
exchange messages. To that effect we’ve implemented mod_msg_filter and
figured we’d share it in case anyone else on the list has similar use
cases.

Here’s the blurb:

mod_msg_filter allows the filtering of "message"
stanzas across an HTTP service. The URL of the
service must be passed as part of the module''s
configuration. Both JIDs and their resources are
passed as part of the query string and the result
is expected to be one of:

<status value="denied">
 <stanza1><error/></stanza1>
 <stanza2><error/></stanza2>
</status>

or:

<status value="allowed">
 <stanza1><noop/></stanza1>
 <stanza2><noop/></stanza2>
</status>

The values of the <error> tags or <noop> tags will
be cached in mnesia using 2 keys that look like:

{bare_jid1, bare_jid2, resource1, resource2}
{bare_jid2, bare_jid1, resource2, resource1}

The <error> tags will then be sent over to both JIDs
if the <status> has a "value" of "denied", otherwise
the original message is let through.

The mnesia cache can be flushed if the ejabberd
server is hit on a request handler that maps to this
module (for example: /mod_msg_filter/). A stanza must
be POSTed that looks like:

<flush jid="user@domain.com">

Note that no resource is included at all. This can be
used if ejabberd is part of a system that has billing
restrictions on chatting but allows presence to go
through all the time.

</status></error></noop></error></pre>

And you can find the code here.

<script src="https://gist.github.com/mardambey/2486170.js"></script>
