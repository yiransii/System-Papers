# System-Papers
Papres I find interesting. Mostly system related.

------------------------------------------------------------------------------
## Nexus: A GPU Cluster Engine for Accelerating DNN-Based Video Analysis


**Source Code**
https://github.com/uwsampl/nexus

**Paper**
https://dl.acm.org/doi/pdf/10.1145/3341301.3359658

**Problem and Motivation**
GPU instances is much more powerful than CPU instances. But existing serving systems do not optimize resources for DNN applications. Fundamental challenge: distribute large workload onto a cluster of accelerators with high utilization and latency requirements. 
Motivation: a DNN serving system that offers 1) high throughput by leveraging GPU hardwares 2) low latency (meeting latency SLO(server level objects) requirements) 3) flexibility(multi-application, auto scaling etc)

**Solution Overview**


Pipeline descriptions:

User upload a model with some information about the model(batch-latency profile) into the system model database. Upon submitting a query with some latency constraint, the system first compute the optimized latency split then determines if model-prefix could be done, then performs squichy bin packing to minimize #GPUs used to compute the query and lower runtime latency while meeting the latency constraint. Then the global scheduler allocated GPUs as needed and sends the query, the model, and all other information to those GPU instances to do the serving. Once a query is completed, the GPU instances talks to the frontend of the system, which returns the result to the user. The system also takes workload data into accounts when managing GPUs via its cluster manager and global scheduler. 

3 main contributions/optimizations:

1) prefixed batching:

identify the common prefix of different models and execute the common prefix as a batch for different inputs. Thus only need to load the common prefix of the model into memory once, which lowers the latency-wise cost

2) complex query analysis via optimized latency partition:
 
optimize latency partition for the sequential computation tasks given a DNN model data graph(tree-like graph) with dynamic programming to minimize the number of GPUs required to complete a query. Specifically, let f(u, t) be the number of GPUs required to complete a query with input u(a given model) and t(the latency requirements). f(u,t) = min( min(#GPU(cur_node_given_latency_requirement)) + min(Sum(#GPU(child_nodes_given_latency_requirement)))  

3) batch-aware resource allocator via squishy bin packing algorithm: 

merge tasks of 2 nodes into 1 node with adjustments made to duty cycle and batch size if the merged node would not overwork. This is to increase GPU utilization of each instance, thus increase the system's throughput. To extend the algorithm system-wide, Nexus does the following: 1)move tasks from the least used instances to other instances 2)release an instance if no longer executing tasks 3)drop the cheapest tasks on an instance if that instance is overworked

**Questions Raised/Potential Improvements**

1. when a model is too large that one GPU could not load it into its memory, we could build a model partition machanism on top of Nexus.
2. how would the above techiniques perform on other ML applications that are not video analysis? Or complex queies that does not result in a tree-structured data flow graph?

------------------------------------------------------------------------------
## PipeDream: Generalized Pipeline Parallelism for DNN Training
**Problem and Motivation**

Existing DNN training systems have improvements that could be made due to the bi-directional pass of DNN — either data pallalelization or model parallelization or hybrid or GPipe all have the problem that intra batch parallelization can suffer from high communication costs at large scale. 

The goal of PipeDream is to achieve high hardware efficiency with no pipeline stalls in steady state, and high statistical efficiency comparable to data parallelism using the same number of workers. 

**Hypothesis**

**Solution Overview**

On a high level, the solution is to purpose a pipeline parallelism system where the DNN algorithms are partitioned into different pipeline stages, where each stage consists of a consecutive set of layers int eh model.  Each stage is mapped to a seperate GPU that performs the forward pass(and backward pass) for all layers in that stage. Each worker(aka GPU in the paper) immediately sends forward activation and backward gradient to one corresponding worker, while simultaneously starting the computation for the next minibatch.

1) 1-Forward-1-Backword algorithm

keep hardware well utilized while achieving semantics similar to data parallelism. Each worker strictly alters between backward and forward passes for its stage. Use different versions of model weights to maintain statistical efficiency.

2) effective load balancing for each stage in the DNN pipeline

automatically partition the operators of the DNN based on a short profiling run performed on a single GPU, then load balances the computational local among the different stages while minimizing communication for the target platform. 

3) 1F1B-RoundRobin

extend 1F1B to incorporates RR scheduling across data-parallel stages while making sure the backward pass worker are routed to the corresponding forward pass worker.

------------------------------------------------------------------------------
## Lineage Stash: Fault Tolerance Off the Critical Path

**Problem and Motivation**

Fault tolerance is growing in importance for cluster computing frameworks. Fault tolerance strategy used by current systems have tradeoffs. Checkpointing exhibits low overhead during normal operation but high overhead during recovery, while lineage-based solutions make the opposite tradeoffs.

The motivation is to develop some technics that would significantly reduces the runtime overhead of lineage-based approaches without impacting recovery efficiency.

**Hypothesis**

Asynchronously record each task's information and forwarding the lineage along with the task will reduce the runtime and recovery overheads for large-scale, low-latency systems.

**Solution Overview**

The solution: Lineage stash - a decentralized causal logging technique. The main idea is to extend the ideas of both lineage reconstruction and causal logging by identifying the nondeterministic events that must be logged for application correctness and design an efficient protocol to store this information off the critical path of execution. 

Each worker asynchronously store the lineage and forward only the most recent part which has not been durably stored yet. In particular, Each worker keeps a lineage stash in local memory containing all tasks that it has seen recently. Then each worker runs a local protocol to flush its stash to a global store, a reliable key-value store that maps task ID to specification. 

To ensure global consistency(recovery consistency), rules are defined for global orderings. The lineage of a task consists of the task itself and the linage of all its dependencies. Lineage consistency after failure is ensured by the following rule: each worker would forward uncommitted lineage with each submitted task. However, only nondeterministic events need to be forwarded to receiving nodes. Deterministic events only need to remember tasks that have been submitted so far.

**Limitations and Possible Improvements**

The recovery protocol can be further optimized by leveraging the property common to decentralized data processing applications: that most processes only send tasks to a small amount of other processes.

------------------------------------------------------------------------------
## Ray: a distributed framework for emerging AI applications

**Problem and Motivation**

Current frameworks do not satisfy the requirements for emerging AI applications. Ray is purposed for the the serving, training, and simulations of reinforcement learning applications, which requires  fine-grained and heterogeneous computations.

**Hypothesis**

Ray's architecture would meet the requirements for latency, scalability and fault tolerance. 

**Solution Overview**

Ray is a distributed framework that supports: fine-grained, heterogeneous computations and dynamic executions. 

deploys a programming model where the basic components are tasks(the execution of a remote function on a stateless worker) and actors(a stateful computation). To meet the requirements for heterogeneity and flexibility, Ray provides specific APIs to have the first K available results returned, to let developers to specify resources requirements, and to enable remote functions revoke other remote functions. 

Architecture wise, Ray is comprised of an application layer implementing the API, and a system layer providing high scalability and fault tolerance. The application layer consists of three types of processes: driver(executes the user program), worker(stateless process initiated by the driver) and actor(stateful process that only executes the method it exposes). The system layer consists of three major components: a global control store(stores the entire control state of the system that enables every component in the system to be stateless), a distributed scheduler(two layer hierarchical scheduler made of local node scheduler and global scheduler), and a distributed object store(...workers in the same node share an in-memory object store).

**Limitations and Possible Improvements**

Specialized optimizations are hard given the workload generality, e.g. scheduling optimizations.

------------------------------------------------------------------------------
## The Hadoop Distributed File System
I like reading this paper because it connects pieces I learned from operating systems class and web systems class together. HDFS has all its stucture designed to imporve data availablity, data reliability, as well as network bandwith utilization. 

It provides interesting examples of HDFS's structure to achieve the above goals. For instance, how HDFS provides consistancy upon read and write request from clients. Just like a non-distributed file system, HDFS implements a single writer and multiple reader model. However, the difference between HDFS and a the non-distributed file system project from EECS482 at Umich is that HDFS provides its client a locking service. A client would sign a renewable lease when trying to write to a node and the client would have exclusive access to a file after signing the lease. The client then needs to renew the lease by sending hearbeat messages periotically. 

