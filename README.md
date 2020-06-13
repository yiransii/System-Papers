# System-Papers
Papres I find interesting. Mostly system related.

------------------------------------------------------------------------------
### The Hadoop Distributed File System
I like reading this paper because it connects pieces I learned from operating systems class and web systems class together. HDFS has all its stucture designed to imporve data availablity, data reliability, as well as network bandwith utilization. 

It provides interesting examples of HDFS's structure to achieve the above goals. For instance, how HDFS provides consistancy upon read and write request from clients. Just like a non-distributed file system, HDFS implements a single writer and multiple reader model. However, the difference between HDFS and a the non-distributed file system project from EECS482 at Umich is that HDFS provides its client a locking service. A client would sign a renewable lease when trying to write to a node and the client would have exclusive access to a file after signing the lease. The client then needs to renew the lease by sending hearbeat messages periotically. 

