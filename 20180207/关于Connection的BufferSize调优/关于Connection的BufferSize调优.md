Poor network utilization for large TCP data transfers is often a symptom
of an overly
small [ConnectionBufferSize](http://blogs.msdn.com/drnick/archive/2006/03/10/547568.aspx).
The ConnectionBufferSize is the size of the send and receive buffers
used by the connection oriented transports, and in particular the TCP
transport where the default size is 8 KB.

If all of the following factors are present, then you might want to
investigate whether tuning the connection buffer size will improve the
performance of your application:

-   Using the TCP transport in a binding

-   Bottleneck is in data transfer rather than data computation

-   Regularly transferring large messages or experiencing high
connection latency

-   Network utilization is noticeably lower than file copies or other
data transfers between the same two machines

On the other hand, the following factors might discourage you from
increasing the connection buffer size:

-   Memory pressure

-   Interactive applications

I've previously [recommended increasing the
ConnectionBufferSize](http://blogs.msdn.com/drnick/archive/2008/02/04/tcp-throttling.aspx) up
to 64 KB once the network speed exceeds 100 Mbps and keeping the default
for slower networks. I've found recently though several deployments
where performance has benefited from bumping up the connection buffer
size more aggressively. The exact optimal values for a deployment depend
on many factors besides the bandwidth, such as the frequency of dropped
packets, transmission latency, and the presence of routers or bridges.

I'm now considering recommended ranges of 8 KB-32 KB for connection up
to 100 Mbps and 32 KB-256 KB for faster connections. If you've tried
tuning the ConnectionBufferSize of a WCF application and seen optimal
values significantly outside these ranges, I'd be interested in hearing
from you.