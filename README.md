# osu-micro-benchmarks
OSU Micro-Benchmark Suite for evaluating MPI, PAPI, and OpenSHMEM communication performance. Developed by the Ohio State University 

OMB (OSU Micro-Benchmarks)
--------------------------
The OSU Micro-Benchmarks use the GNU build system. Therefore you can simply
use the following steps to build the MPI benchmarks.

Example:
	./configure CC=/path/to/mpicc CXX=/path/to/mpicxx
	make
	make install

CC and CXX can be set to other wrapper scripts as well to build OpenSHMEM or
UPC++ benchmarks as well.  Based on this setting, configure will detect whether
your library supports MPI-1, MPI-2, MPI-3, OpenSHMEM, and UPC++ to compile the
corresponding benchmarks.  See http://mvapich.cse.ohio-state.edu/benchmarks/ to
download the latest version of this package.

The python directory contains Python versions of the benchmarks. These do not
need to be compiled. See the README file located in the python directory
for more details.

The Java directory contains Java versions of the benchmarks. These are compiled
separately via the javac compiler. See the README file located in the java
directory for more details.

OMB also contains ROCm, CUDA, SYCL and OpenACC extensions to the benchmarks.
CUDA extensions can be enabled by configuring OMB with --enable-cuda option as
shown below.

    ./configure CC=/path/to/mpicc
                CXX=/path/to/mpicxx
                --enable-cuda
                --with-cuda-include=/path/to/cuda/include
                --with-cuda-libpath=/path/to/cuda/lib
    make
    make install

ROCm extensions can be enabled by configuring OMB with --enable-rocm
option as shown below. Similarly OpenACC extensions can be enabled using
--enable-openacc option. The MPI library used should be able to support MPI
communication from buffers in GPU Device memory.

    ./configure CC=/path/to/mpicc
                CXX=/path/to/mpicxx
                --enable-rocm
                --with-rocm=/path/to/rocm/install
    make
    make install

SYCL extensions can be enabled by configuring OMB with --enable-sycl
option as shown below.

    ./configure CC=/path/to/mpicc
                CXX=/path/to/mpicxx
                --enable-sycl
                --with-sycl=/path/to/sycl/install
    make
    make install


More information about the ROCm and CUDA extensions are given towards the end of
the README.

This package also distributes UPC put, get, and collective benchmarks.
These are located in the upc subdirectory and can be compiled by the
following:

        for bench in osu_upc_memput              \
                     osu_upc_memget              \
                     osu_upc_all_scatter         \
                     osu_upc_all_reduce          \
                     osu_upc_all_gather          \
                     osu_upc_all_gather_all      \
                     osu_upc_all_exchange        \
                     osu_upc_all_broadcast       \
                     osu_upc_all_barrier
        do
            echo "Compiling $bench..."
            upcc $bench.c ../util/osu_util_pgas.c ../util/osu_util.c -o $bench
        done

The MPI Multiple Bandwidth / Message Rate (osu_mbw_mr), OpenSHMEM Put Message
Rate (osu_oshm_put_mr), and OpenSHMEM Atomics (osu_oshm_atomics) tests are
intended to be used with block assigned ranks.  This means that all processes
on the same machine are assigned ranks sequentially.

Rank	Block   Cyclic
----------------------
0	host1	host1
1	host1	host2
2	host1	host1
3	host1	host2
4	host2	host1
5	host2	host2
6	host2	host1
7	host2	host2

If you're using mpirun_rsh the ranks are assigned in the order they are seen in
the hostfile or on the command line.  Please see your process managers'
documentation for information on how to control the distribution of the rank to
host mapping.

Point-to-Point MPI Benchmarks
-----------------------------
osu_latency - Latency Test
    * The latency tests are carried out in a ping-pong fashion. The sender
    * sends a message with a certain data size to the receiver and waits for a
    * reply from the receiver. The receiver receives the message from the sender
    * and sends back a reply with the same data size. Many iterations of this
    * ping-pong test are carried out and average one-way latency numbers are
    * obtained. Blocking version of MPI functions (MPI_Send and MPI_Recv) are
    * used in the tests. This test is available here.

osu_latency_mt - Multi-threaded Latency Test
    * The multi-threaded latency test performs a ping-pong test with a single
    * sender process and multiple threads on the receiving process. In this test
    * the sending process sends a message of a given data size to the receiver
    * and waits for a reply from the receiver process. The receiving process has
    * a variable number of receiving threads (set by default to 2), where each
    * thread calls MPI_Recv and upon receiving a message sends back a response
    * of equal size. Many iterations are performed and the average one-way
    * latency numbers are reported. This test is available here.
    * "-t" option can be used to set the number of sender and receiver threads
           to be used in a benchmark. Examples:
            -t 4        // receiver threads = 4 and sender threads = 1
            -t 4:6      // sender threads = 4 and receiver threads = 6
            -t 2:       // not defined

osu_latency_mp - Multi-process Latency Test
    * The multi-process latency test performs a ping-pong test with a single
    * sender process and a single receiver process, both having one or more
    * child processes that are spawned using the fork() system call. In this test
    * the sending process(parent) sends a message of a given data size to the
    * receiver(parent) process and waits for a reply from the receiver process.
    * Both the sending and receiving process have a variable number of child
    * processes (set by default to 1 child process), where each child process
    * sleeps for 2 seconds after the fork call and exits. The parent processes
    * carry out the ping-pong test where many iterations are performed and the
    * average one-way latency numbers are reported. This test is available here.
    * "-t" option can be used to set the number of sender and receiver processes
    * including the parent processes to be used in a benchmark.
    *
    * The purpose of this test is to check if the underlying MPI communication
    * runtime has taken care of fork safety even if the application has not.
    *
    * A new environment variable "MV2_SUPPORT_FORK_SAFETY" was introduced with
    * MVAPICH2 2.3.4 to make MVAPICH2 takes care of fork safety for
    * applications that require it.
    *
    * The support for fork safety is disabled by default in MVAPICH2 due to
    * performance reasons. When running osu_latency_mp with MVAPICH2, set
    * the environment variable MV2_SUPPORT_FORK_SAFETY to 1. When running
    * osu_latency_mp with other MPI libraries that do not support fork safety,
    * set the environment variables RDMAV_FORK_SAFE or IBV_FORK_SAFE to 1.
    * Examples:
            -t 4        // receiver processes = 4 and sender processes = 1
            -t 4:6      // sender processes = 4 and receiver processes = 6
            -t 2:       // not defined

osu_bw - Bandwidth Test
    * The bandwidth tests are carried out by having the sender sending out a
    * fixed number (equal to the window size) of back-to-back messages to the
    * receiver and then waiting for a reply from the receiver. The receiver
    * sends the reply only after receiving all these messages. This process is
    * repeated for several iterations and the bandwidth is calculated based on
    * the elapsed time (from the time sender sends the first message until the
    * time it receives the reply back from the receiver) and the number of bytes
    * sent by the sender. The objective of this bandwidth test is to determine
    * the maximum sustained date rate that can be achieved at the network level.
    * Thus, non-blocking version of MPI functions (MPI_Isend and MPI_Irecv) are
    * used in the test. This test is available here.

osu_bibw - Bidirectional Bandwidth Test
    * The bidirectional bandwidth test is similar to the bandwidth test, except
    * that both the nodes involved send out a fixed number of back-to-back
    * messages and wait for the reply. This test measures the maximum
    * sustainable aggregate bandwidth by two nodes. This test is available here.

osu_mbw_mr - Multiple Bandwidth / Message Rate Test
    * The multi-pair bandwidth and message rate test evaluates the aggregate
    * uni-directional bandwidth and message rate between multiple pairs of
    * processes. Each of the sending processes sends a fixed number of messages
    * (the window size) back-to-back to the paired receiving process before
    * waiting for a reply from the receiver. This process is repeated for
    * several iterations. The objective of this benchmark is to determine the
    * achieved bandwidth and message rate from one node to another node with a
    * configurable number of processes running on each node. The test is
    * available here.

osu_multi_lat - Multi-pair Latency Test
    * This test is very similar to the latency test. However, at the same
    * instant multiple pairs are performing the same test simultaneously.
    * In order to perform the test across just two nodes the hostnames must
    * be specified in block fashion.

Building XCCL(NCCL/RCCL) benchmarks
--------------------------------------
NCCL and RCCL benchmarks are now merged into XCCL benchmarks. XCCL benchmarks
will run with either NCCL or RCCL support set during configure time.

Configure command line to enable NCCL benchmarks is as follows:
--enable-cuda --with-cuda-include=/path/to/cuda/include
--with-cuda-libpath=/path/to/cuda/lib64 --enable-ncclomb
--with-nccl=/path/to/nccl/build

Configure command line to enable RCCL benchmarks is as follows:
--enable-rocm --with-rocm=/path/to/rocm/build --enable-rcclomb
--with-rccl=/path/to/rocm/build

Point-to-Point XCCL(NCCL/RCCL) Benchmarks
------------------------------
osu_latency - Latency Test
    * The latency tests are carried out in a ping-pong fashion. The sender
    * sends a message with a certain data size to the receiver and waits for a
    * reply from the receiver. The receiver receives the message from the sender
    * and sends back a reply with the same data size. Many iterations of this
    * ping-pong test are carried out and average one-way latency numbers are
    * obtained. Non-Blocking version of XCCL functions (ncclSend and ncclRecv)
    * are used in the tests. This test is available here.

osu_bw - Bandwidth Test
    * The bandwidth tests are carried out by having the sender sending out a
    * fixed number (equal to the window size) of back-to-back messages to the
    * receiver and then waiting for a reply from the receiver. The receiver
    * sends the reply only after receiving all these messages. This process is
    * repeated for several iterations and the bandwidth is calculated based on
    * the elapsed time (from the time sender sends the first message until the
    * time it receives the reply back from the receiver) and the number of bytes
    * sent by the sender. The objective of this bandwidth test is to determine
    * the maximum sustained date rate that can be achieved at the network level.
    * Thus, non-blocking version of XCCL functions (ncclSend and ncclRecv) are
    * used in the test. This test is available here.

osu_bibw - Bidirectional Bandwidth Test
    * The bidirectional bandwidth test is similar to the bandwidth test, except
    * that both the nodes involved send out a fixed number of back-to-back
    * messages and wait for the reply. This test measures the maximum
    * sustainable aggregate bandwidth by two nodes. This test is available here.

Collective MPI Benchmarks
-------------------------
osu_allgather      - MPI_Allgather Latency Test(*)
osu_allgatherv     - MPI_Allgatherv Latency Test
osu_allreduce      - MPI_Allreduce Latency Test
osu_alltoall       - MPI_Alltoall Latency Test
osu_alltoallv      - MPI_Alltoallv Latency Test
osu_barrier        - MPI_Barrier Latency Test
osu_bcast          - MPI_Bcast Latency Test
osu_gather         - MPI_Gather Latency Test(*)
osu_gatherv        - MPI_Gatherv Latency Test
osu_reduce         - MPI_Reduce Latency Test
osu_reduce_scatter - MPI_Reduce_scatter Latency Test
osu_scatter        - MPI_Scatter Latency Test(*)
osu_scatterv       - MPI_Scatterv Latency Test

Collective Latency Tests
    * The latest OMB version includes benchmarks for various MPI blocking
    * collective operations (MPI_Allgather, MPI_Alltoall, MPI_Allreduce,
    * MPI_Barrier, MPI_Bcast, MPI_Gather, MPI_Reduce, MPI_Reduce_Scatter,
    * MPI_Scatter and vector collectives). These benchmarks work in the
    * following manner.  Suppose users run the osu_bcast benchmark with N
    * processes, the benchmark measures the min, max and the average latency of
    * the MPI_Bcast collective operation across N processes, for various
    * message lengths, over a large number of iterations. In the default
    * version, these benchmarks report the average latency for each message
    * length. Additionally, the benchmarks offer the following options:
    * "-f" can be used to report additional statistics of the benchmark,
           such as min and max latencies and the number of iterations.
    * "-m" option can be used to set the minimum and maximum message length
           to be used in a benchmark. In the default version, the benchmarks
           report the latencies for up to 1MB message lengths. Examples:
            -m 128      // min = default, max = 128
            -m 2:128    // min = 2, max = 128
            -m 2:       // min = 2, max = default
    * "-x" can be used to set the number of warmup iterations to skip for each
           message length.
    * "-i" can be used to set the number of iterations to run for each message
           length.

Collective XCCL(NCCL/RCCL) Benchmarks
--------------------------
osu_xccl_allgather      - XCCL Allgather Latency Test(*)
osu_xccl_allreduce      - XCCL Allreduce Latency Test
osu_xccl_alltoall       - XCCL Alltoall Latency Test
osu_xccl_bcast          - XCCL Bcast Latency Test
osu_xccl_reduce         - XCCL Reduce Latency Test
osu_xccl_reduce_scatter - XCCL Reduce_scatter Latency Test

Collective Latency Tests
    * The latest OMB version includes benchmarks for various MPI blocking
    * collective operations (NCCL Allgather, NCCL Allreduce, NCCL Alltoall,
    * NCCL Bcast, NCCL Reduce, NCCL Reduce_Scatter. These benchmarks work in the
    * following manner.  Suppose users run the osu_xccl_bcast benchmark with N
    * processes, the benchmark measures the min, max and the average latency of
    * the NCCL Bcast collective operation across N processes, for various
    * message lengths, over a large number of iterations. In the default
    * version, these benchmarks report the average latency for each message
    * length. Additionally, the benchmarks offer the following options:
    * "-f" can be used to report additional statistics of the benchmark,
           such as min and max latencies and the number of iterations.
    * "-m" option can be used to set the minimum and maximum message length
           to be used in a benchmark. In the default version, the benchmarks
           report the latencies for up to 1MB message lengths. Examples:
            -m 128      // min = default, max = 128
            -m 2:128    // min = 2, max = 128
            -m 2:       // min = 2, max = default
    * "-x" can be used to set the number of warmup iterations to skip for each
           message length.
    * "-i" can be used to set the number of iterations to run for each message
           length.

Support for Data Validation
---------------------------
The following benchmarks have been extended to evalaute the correctness in
addition to the communication performance.

    * osu_allreduce         - MPI_Allreduce Latency Test
    * osu_xccl_allreduce    - XCCL Allreduce Latency Test
    * osu_reduce            - MPI_Reduce Latency Test
    * osu_xccl_reduce       - XCCL Reduce Latency Test
    * osu_alltoall          - MPI_Alltoall Latency Test

Support for CUDA Managed Memory
-------------------------------
The following benchmarks have been extended to evaluate performance of MPI communication
from and to buffers allocated using CUDA Managed Memory.

    * osu_bibw              - Bidirectional Bandwidth Test
    * osu_bw                - Bandwidth Test
    * osu_latency           - Latency Test
    * osu_mbw_mr            - Multiple Bandwidth / Message Rate Test
    * osu_multi_lat         - Multi-pair Latency Test
    * osu_allgather         - MPI_Allgather Latency Test
    * osu_allgatherv        - MPI_Allgatherv Latency Test
    * osu_allreduce         - MPI_Allreduce Latency Test
    * osu_alltoall          - MPI_Alltoall Latency Test
    * osu_alltoallv         - MPI_Alltoallv Latency Test
    * osu_bcast             - MPI_Bcast Latency Test
    * osu_gather            - MPI_Gather Latency Test
    * osu_gatherv           - MPI_Gatherv Latency Test
    * osu_reduce            - MPI_Reduce Latency Test
    * osu_reduce_scatter    - MPI_Reduce_scatter Latency Test
    * osu_scatter           - MPI_Scatter Latency Test
    * osu_scatterv          - MPI_Scatterv Latency Test

In addition to support for communications to and from GPU memories allocated
using CUDA or OpenACC, we now provide additional capability of performing
communications to and from buffers allocated using the CUDA Managed Memory concept.
CUDA Managed (or Unified) Memory allows applications to allocate memory on either CPU
or GPU memories using the cudaMallocManaged() call. This allows user oblivious transfer
of the memory buffer between the CPU or GPU. Currently, we offer benchmarking with CUDA
Managed Memory using the tests mentioned above.

These benchmarks have additional options:
    * "M" allocates a send or receive buffer as managed for point to point communication.
    * "-d managed" uses managed memory buffers to perform collective communications.


Non-Blocking Collective MPI Benchmarks
--------------------------------------
osu_iallgather    - MPI_Iallgather Latency Test
osu_iallgatherv   - MPI_Iallgatherv Latency Test
osu_iallreduce    - MPI_Iallreduce Latency Test
osu_ialltoall     - MPI_Ialltoall Latency Test
osu_ialltoallv    - MPI_Ialltoallv Latency Test
osu_ialltoallw    - MPI_Ialltoallw Latency Test
osu_ibarrier      - MPI_Ibarrier Latency Test
osu_ibcast        - MPI_Ibcast Latency Test
osu_igather       - MPI_Igather Latency Test
osu_igatherv      - MPI_Igatherv Latency Test
osu_ireduce       - MPI_Ireduce Latency Test
osu_iscatter      - MPI_Iscatter Latency Test
osu_iscatterv     - MPI_Iscatterv Latency Test

Non-Blocking Collective Latency Tests
    * In addition to the blocking collective latency tests, we provide several
    * non-blocking collectives as mentioned above. These evaluate the same
    * metrics as the blocking operations as well as the additional metric
    * `overlap'.  This is defined as the amount of computation that can be
    * performed while the communication progresses in the background.
    * These benchmarks have the additional option:
    * "-t" set the number of MPI_Test() calls during the dummy computation, set
           CALLS to 100, 1000, or any number > 0.


One-sided MPI Benchmarks
------------------------
osu_put_latency - Latency Test for Put with Active/Passive Synchronization
    * The put latency benchmark includes window initialization operations
    * (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic) and
    * synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the origin process calls MPI_Put to directly place data of a certain size
    * in the remote process's window and then waiting on a synchronization call
    * (MPI_Win_complete) for completion.  The remote process participates in
    * synchronization with MPI_Win_post and MPI_Win_wait calls. Several
    * iterations of this test is carried out and the average put latency
    * numbers is reported. The latency includes the synchronization time also.
    * For passive synchronization, suppose users run with MPI_Win_lock/unlock,
    * the origin process calls MPI_Win_lock to lock the target process's window
    * and calls MPI_Put to directly place data of certain size in the window.
    * Then it calls MPI_Win_unlock to ensure completion of the Put and release
    * lock on the window. This is carried out for several iterations and the
    * average time for MPI_Lock + MPI_Put + MPI_Unlock calls is measured. The
    * default window initialization and synchronization operations are
    * MPI_Win_allocate and MPI_Win_flush. The benchmark offers the following
    * options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.
    * "-x"              can be used to set the number of warmup iterations to
                        skip for each message length.
    * "-i"              can be used to set the number of iterations to run for
                        each message length.

osu_get_latency - Latency Test for Get with Active/Passive Synchronization
    * The get latency benchmark includes window initialization operations
    * (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic) and
    * synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the origin process calls MPI_Get to directly fetch data of a certain size
    * from the target process's window into a local buffer. It then waits on a
    * synchronization call (MPI_Win_complete) for local completion of the Gets.
    * The remote process participates in synchronization with MPI_Win_post and
    * MPI_Win_wait calls. Several iterations of this test is carried out and
    * the average get latency numbers is reported. The latency includes the
    * synchronization time also. For passive synchronization, suppose users run
    * with MPI_Win_lock/unlock, the origin process calls MPI_Win_lock to lock
    * the target process's window and calls MPI_Get to directly read data of
    * certain size from the window. Then it calls MPI_Win_unlock to ensure
    * completion of the Get and releases lock on remote window. This is carried
    * out for several iterations and the average time for MPI_Lock + MPI_Get +
    * MPI_Unlock calls is measured.  The default window initialization and
    * synchronization operations are MPI_Win_allocate and MPI_Win_flush. The
    * benchmark offers the following options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate "    use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_put_bw - Bandwidth Test for Put with Active/Passive Synchronization
    * The put bandwidth benchmark includes window initialization operations
    * (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic) and
    * synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the test is carried out by the origin process calling a fixed number of
    * back-to-back MPI_Puts on remote window and then waiting on a
    * synchronization call (MPI_Win_complete) for their completion. The remote
    * process participates in synchronization with MPI_Win_post and
    * MPI_Win_wait calls. This process is repeated for several iterations and
    * the bandwidth is calculated based on the elapsed time and the number of
    * bytes put by the origin process. For passive synchronization, suppose
    * users run with MPI_Win_lock/unlock, the origin process calls MPI_Win_lock
    * to lock the target process's window and calls a fixed number of
    * back-to-back MPI_Puts to directly place data in the window. Then it calls
    * MPI_Win_unlock to ensure completion of the Puts and release lock on
    * remote window. This process is repeated for several iterations and the
    * bandwidth is calculated based on the elapsed time and the number of bytes
    * put by the origin process. The default window initialization and
    * synchronization operations are MPI_Win_allocate and MPI_Win_flush.  The
    * benchmark offers the following options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_get_bw - Bandwidth Test for Get with Active/Passive Synchronization
    * The get bandwidth benchmark includes window initialization operations
    * (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic) and
    * synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the test is carried out by origin process calling a fixed number of
    * back-to-back MPI_Gets and then waiting on a synchronization call
    * (MPI_Win_complete) for their completion. The remote process participates
    * in synchronization with MPI_Win_post and MPI_Win_wait calls. This process
    * is repeated for several iterations and the bandwidth is calculated based
    * on the elapsed time and the number of bytes received by the origin
    * process. For passive synchronization, suppose users run with
    * MPI_Win_lock/unlock, the origin process calls MPI_Win_lock to lock the
    * target process's window and calls a fixed number of back-to-back MPI_Gets
    * to directly get data from the window. Then it calls MPI_Win_unlock to
    * ensure completion of the Gets and release lock on the window. This
    * process is repeated for several iterations and the bandwidth is
    * calculated based on the elapsed time and the number of bytes read by the
    * origin process.  The default window initialization and synchronization
    * operations are MPI_Win_allocate and MPI_Win_flush. The benchmark offers
    * the following options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization.

osu_put_bibw - Bi-directional Bandwidth Test for Put with Active
               Synchronization
    * The put bi-directional bandwidth benchmark includes window initialization
    * operations (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic)
    * and synchronization operations (MPI_Win_Post/Start/Complete/Wait and
    * MPI_Win_fence).  This test is similar to the bandwidth test, except that
    * both the processes involved send out a fixed number of back-to-back
    * MPI_Puts and wait for their completion. This test measures the maximum
    * sustainable aggregate bandwidth by two processes. The default window
    * initialization and synchronization operations are MPI_Win_allocate and
    * MPI_Win_Post/Start/Complete/Wait. The benchmark offers the following
    * options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_acc_latency - Latency Test for Accumulate with Active/Passive
                  Synchronization
    * The accumulate latency benchmark includes window initialization
    * operations (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic)
    * and synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the origin process calls MPI_Accumulate to combine data from the local
    * buffer with the data in the remote window and store it in the remote
    * window. The combining operation used in the test is MPI_SUM. The origin
    * process then waits on a synchronization call (MPI_Win_complete) for
    * completion of the operations. The remote process waits on a MPI_Win_wait
    * call. Several iterations of this test are carried out and the average
    * accumulate latency number is obtained. The latency includes the
    * synchronization time also.  For passive synchronization, suppose users
    * run with MPI_Win_lock/unlock, the origin process calls MPI_Win_lock to
    * lock the target process's window and calls MPI_Accumulate to combine data
    * from a local buffer with the data in the remote window and store it in
    * the remote window.  Then it calls MPI_Win_unlock to ensure completion of
    * the Accumulate and release lock on the window. This is carried out for
    * several iterations and the average time for MPI_Lock + MPI_Accumulate +
    * MPI_Unlock calls is measured. The default window initialization and
    * synchronization operations are MPI_Win_allocate and MPI_Win_flush. The
    * benchmark offers the following options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_cas_latency - Latency Test for Compare and Swap with Active/Passive
                  Synchronization
    * The Compare_and_swap latency benchmark includes window initialization
    * operations (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic)
    * and synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with
    * MPI_Win_Post/Start/Complete/Wait,the origin process calls
    * MPI_Compare_and_swap to place one element from  origin buffer to target
    * buffer.  The initial value in the target buffer is returned to the
    * calling process. The origin process then waits on a synchronization call
    * (MPI_Win_complete) for local completion of the operations. The remote
    * process waits on a MPI_Win_wait call. Several iterations of this test are
    * carried out and the average Compare_and_swap latency number is obtained.
    * The latency includes the synchronization time also.  For passive
    * synchronization, suppose users run with MPI_Win_lock/unlock, the origin
    * process calls MPI_Win_lock to lock the target process's window and calls
    * MPI_Compare_and_swap to place one element from  origin buffer to target
    * buffer. The initial value in the target buffer is returned to the calling
    * process. Then it calls MPI_Win_flush to ensure completion of the
    * Compare_and_swap. In the end, it calls MPI_Win_unlock to release lock on
    * the window. This is carried out for several iterations and the average
    * time for MPI_Compare_and_swap + MPI_Win_flush calls is measured. The
    * default window initialization and synchronization operations are
    * MPI_Win_allocate and MPI_Win_flush. The benchmark offers the following
    * options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_fop_latency - Latency Test for Fetch and Op with Active/Passive
                  Synchronization
    * The Fetch_and_op latency benchmark includes window initialization
    * operations (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic)
    * and synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the origin process calls MPI_Fetch_and_op to increase the element in
    * target buffer by 1. The initial value from the target buffer is returned
    * to the calling process. The origin process waits on a synchronization
    * call (MPI_Win_complete) for completion of the operations. The remote
    * process waits on a MPI_Win_wait call. Several iterations of this test are
    * carried out and the average Fetch_and_op latency number is obtained. The
    * latency includes the synchronization time also.  For passive
    * synchronization, suppose users run with MPI_Win_lock/unlock, the origin
    * process calls MPI_Win_lock to lock the target process's window and calls
    * MPI_Compare_and_swap to place one element from  origin buffer to target
    * buffer. The initial value in the target buffer is returned to the calling
    * process. Then it calls MPI_Win_flush to ensure completion of the
    * Compare_and_swap. In the end, it calls MPI_Win_unlock to release lock on
    * the window. This is carried out for several iterations and the average
    * time for MPI_Compare_and_swap + MPI_Win_flush calls is measured. The
    * default window initialization and synchronization operations are
    * MPI_Win_allocate and MPI_Win_flush. The benchmark offers the following
    * options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

osu_get_acc_latency - Latency Test for Get_accumulate with Active/Passive
                      Synchronization
    * The Get_accumulate latency benchmark includes window initialization
    * operations (MPI_Win_create, MPI_Win_allocate and MPI_Win_create_dynamic)
    * and synchronization operations (MPI_Win_lock/unlock, MPI_Win_flush,
    * MPI_Win_flush_local, MPI_Win_lock_all/unlock_all,
    * MPI_Win_Post/Start/Complete/Wait and MPI_Win_fence). For active
    * synchronization, suppose users run with MPI_Win_Post/Start/Complete/Wait,
    * the origin process calls MPI_Get_accumulate to combine data from the
    * local buffer with the data in the remote window and store it in the
    * remote window. The combining operation used in the test is MPI_SUM. The
    * initial value from the target buffer is returned to the calling process.
    * The origin process waits on a synchronization call (MPI_Win_complete) for
    * local completion of the operations. The remote process waits on a
    * MPI_Win_wait call. Several iterations of this test are carried out and
    * the average get accumulate latency number is obtained. The latency
    * includes the synchronization time also.  For passive synchronization,
    * suppose users run with MPI_Win_lock/unlock, the origin process calls
    * MPI_Win_lock to lock the target process's window and calls
    * MPI_Get_accumulate to combine data from a local buffer with the data in
    * the remote window and store it in the remote window.  The initial value
    * from the target buffer is returned to the calling process.  Then it calls
    * MPI_Win_unlock to ensure completion of the Get_accumulate and release
    * lock on the window. This is carried out for several iterations and the
    * average time for MPI_Lock + MPI_Get_accumulate + MPI_Unlock calls is
    * measured. The default window initialization and synchronization
    * operations are MPI_Win_allocate and MPI_Win_flush. The benchmark offers
    * the following options:
    * "-w create"       use MPI_Win_create to create an MPI Window object.
    * "-w allocate"     use MPI_Win_allocate to create an MPI Window object.
    * "-w dynamic"      use MPI_Win_create_dynamic to create an MPI Window
    *                   object.
    * "-s lock"         use MPI_Win_lock/unlock synchronizations calls.
    * "-s flush"        use MPI_Win_flush synchronization call.
    * "-s flush_local"  use MPI_Win_flush_local synchronization call.
    * "-s lock_all"     use MPI_Win_lock_all/unlock_all synchronization calls.
    * "-s pscw"         use Post/Start/Complete/Wait synchronization calls.
    * "-s fence"        use MPI_Win_fence synchronization call.

Point-to-Point OpenSHMEM Benchmarks
-----------------------------------
osu_oshm_put.c - Latency Test for OpenSHMEM Put Routine
    * This benchmark measures latency of a shmem putmem operation for different
    * data sizes. The user is required to select whether the communication
    * buffers should be allocated in global memory or heap memory, through a
    * parameter. The test requires exactly two PEs. PE 0 issues shmem putmem to
    * write data at PE 1 and then calls shmem quiet. This is repeated for a
    * fixed number of iterations, depending on the data size. The average
    * latency per iteration is reported. A few warm-up iterations are run
    * without timing to ignore any start-up overheads.  Both PEs call shmem
    * barrier all after the test for each message size.

osu_oshm_put_bw.c - Bandwidth Test for OpenSHMEM Put Routine
    * This benchmark is similar to osu_oshm_put.c except it measures
    * the total bandwidth (MB/s) instead of latency.

osu_oshm_put_nb.c - Latency Test for OpenSHMEM Non-blocking Put Routine
    * This benchmark measures the non-blocking latency of a shmem putmem_nbi
    * operation for different data sizes. The user is required to select
    * whether the communication buffers should be allocated in global
    * memory or heap memory, through a parameter. The test requires exactly
    * two PEs. PE 0 issues shmem putmem_nbi to write data at PE 1 and then calls
    * shmem quiet. This is repeated for a fixed number of iterations, depending
    * on the data size. The average latency per iteration is reported.
    * A few warm-up iterations are run without timing to ignore any start-up
    * overheads. Both PEs call shmem barrier all after the test for each message size.

osu_oshm_put_nb_bw.c - Bandwidth Test for OpenSHMEM Non-blocking Put Routine
    * This benchmark is similar to osu_oshm_put_nb.c except it measures
    * the total bandwidth (MB/s) instead of latency.

osu_oshm_get.c - Latency Test for OpenSHMEM Get Routine
    * This benchmark is similar to the one above except that PE 0 does a shmem
    * getmem operation to read data from PE 1 in each iteration. The average
    * latency per iteration is reported.

osu_oshm_get_bw.c - Bandwidth Test for OpenSHMEM Get Routine
    * This benchmark is similar to osu_oshm_get.c except it measures
    * the total bandwidth (MB/s) instead of latency.

osu_oshm_get_nb.c - Latency Test for OpenSHMEM Non-blocking Get Routine
    * This benchmark is similar to the one above except that PE 0 does a shmem
    * getmem_nbi operation to read data from PE 1 in each iteration. The average
    * latency per iteration is reported.

osu_oshm_get_nb_bw.c - Bandwidth Test for OpenSHMEM Non-blocking Get Routine
    * This benchmark is similar to osu_oshm_get_nb.c except it measures
    * the total bandwidth (MB/s) instead of latency.

osu_oshm_put_mr.c - Message Rate Test for OpenSHMEM Put Routine
    * This benchmark measures the aggregate uni-directional operation rate of
    * OpenSHMEM Put between pairs of PEs, for different data sizes. The user
    * should select for communication buffers to be in global memory and heap
    * memory as with the earlier benchmarks. This test requires number of PEs
    * to be even. The PEs are paired with PE 0 pairing with PE n/2 and so on,
    * where n is the total number of PEs. The first PE in each pair issues
    * back-to-back shmem putmem operations to its peer PE. The total time for
    * the put operations is measured and operation rate per second is reported.
    * All PEs call shmem barrier all after the test for each message size.

osu_oshm_put_mr_nb.c - Message Rate Test for Non-blocking OpenSHMEM Put Routine
    * This benchmark measures the aggregate uni-directional operation rate of
    * OpenSHMEM Non-blocking Put between pairs of PEs, for different data sizes.
    * The user should select for communication buffers to be in global memory
    * and heap memory as with the earlier benchmarks. This test requires number
    * of PEs to be even. The PEs are paired with PE 0 pairing with PE n/2 and so on,
    * where n is the total number of PEs. The first PE in each pair issues
    * back-to-back shmem putmem_nbi operations to its peer PE until the window
    * size. A call to shmem_quite is placed after the window loop to ensure
    * completion of the issued operations. The total time for the non-blocking
    * put operations is measured and operation rate per second is reported.
    * All PEs call shmem barrier all after the test for each message size.

osu_oshm_get_mr_nb.c - Message Rate Test for Non-blocking OpenSHMEM Get Routine
    * This benchmark measures the aggregate uni-directional operation rate of
    * OpenSHMEM Non-blocking Get between pairs of PEs, for different data sizes.
    * The user should select for communication buffers to be in global memory
    * and heap memory as with the earlier benchmarks. This test requires number
    * of PEs to be even. The PEs are paired with PE 0 pairing with PE n/2 and so on,
    * where n is the total number of PEs. The first PE in each pair issues
    * back-to-back shmem getmem_nbi operations to its peer PE until the window
    * size. A call to shmem_quite is placed after the window loop to ensure
    * completion of the issued operations. The total time for the non-blocking
    * put operations is measured and operation rate per second is reported.
    * All PEs call shmem barrier all after the test for each message size.

osu_oshm_put_overlap.c - Non-blocking Message Rate Overlap Test
    * This benchmark measures the aggregate uni-directional operations rate
    * overlap for OpenSHMEM Put between paris of PEs, for different data sizes.
    * The user should select for communication buffers to be in global memory
    * and heap memory as with the earlier benchmarks. This test requires number
    * of PEs. The benchmarks prints statistics for different phases of
    * communication, computation and overlap in the end.

osu_oshm_atomics.c - Latency and Operation Rate Test for OpenSHMEM Atomics Routines
    * This benchmark measures the performance of atomic fetch-and-operate and
    * atomic operate routines supported in OpenSHMEM for the integer
    * and long datatypes. The buffers can be selected to be in heap memory or global
    * memory. The PEs are paired like in the case of Put Operation Rate
    * benchmark and the first PE in each pair issues back-to-back atomic
    * operations of a type to its peer PE. The average latency per atomic
    * operation and the aggregate operation rate are reported.  This is
    * repeated for each of fadd, finc, add, inc, cswap, swap, set, and fetch
    * routines.

Collective OpenSHMEM Benchmarks
-------------------------------
osu_oshm_collect   - OpenSHMEM Collect Latency Test
osu_oshm_fcollect  - OpenSHMEM FCollect Latency Test
osu_oshm_broadcast - OpenSHMEM Broadcast Latency Test
osu_oshm_reduce    - OpenSHMEM Reduce Latency Test
osu_oshm_barrier   - OpenSHMEM Barrier Latency Test

Collective Latency Tests
    * The latest OMB Version includes benchmarks for various OpenSHMEM
    * collective operations (shmem_collect, shmem_broadcast, shmem_reduce and
    * shmem_barrier). These benchmarks work in the following manner. Suppose
    * users run the osu_oshm_broadcast benchmark with N processes, the
    * benchmark measures the min, max and the average latency of the
    * shmem_broadcast collective operation across N processes, for various
    * message lengths, over a large number of iterations. In the default
    * version, these benchmarks report the average latency for each message
    * length. Additionally, the benchmarks offer the following options:
    * "-f" can be used to report additional statistics of the benchmark,
           such as min and max latencies and the number of iterations.
    * "-m" option can be used to set the maximum message length to be used in a
           benchmark. In the default version, the benchmarks report the
           latencies for up to 1MB message lengths.
    * "-i" can be used to set the number of iterations to run for each message
           length.

Point-to-Point UPC Benchmarks
-----------------------------
osu_upc_memput.c - Put Latency
    * This benchmark measures the latency of upc put operation between multiple
    * UPC threads. In this bench- mark, UPC threads with ranks less than
    * (THREADS/2) issues upc memput operations to peer UPC threads. Peer
    * threads are identified as (MYTHREAD+THREADS/2). This is repeated for a
    * fixed number of iterations, for varying data sizes. The average latency
    * per iteration is reported. A few warm-up iterations are run without
    * timing to ignore any start-up overheads. All UPC threads call upc barrier
    * after the test for each message size.

osu_upc_memget.c - Get Latency
    * This benchmark is similar as the osu upc put benchmark that is described
    * above. The difference is that the shared string handling function is upc
    * memget. The average get operation latency per iteration is reported.

Collective UPC Benchmarks
-------------------------
osu_upc_all_barrier     - UPC Barrier Latency Test
osu_upc_all_broadcast   - UPC Broadcast Latency Test
osu_upc_all_scatter     - UPC Scatter Latency Test
osu_upc_all_gather      - UPC Gather Latency Test
osu_upc_all_gather_all  - UPC GatherAll Latency Test
osu_upc_all_reduce      - UPC Reduce Latency Test
osu_upc_all_exchange    - UPC Exchange Latency Test

Collective Latency Tests
    * The latest OMB Version includes benchmarks for various UPC collective
    * operations (upc_all_barrier, upc_all_broadcast, upc_all_scatter,
    * upc_all_gather, upc_all_gather_all, osu_upc_all_reduce, and
    * upc_all_exchange). These benchmarks work in the following manner. Suppose
    * users run the osu_upc_all_broadcast benchmark with N processes, the
    * benchmark measures the min, max and the average latency of the
    * upc_all_broadcast collective operation across N processes, for various
    * message lengths, over a large number of iterations. In the default
    * version, these benchmarks report the average latency for each message
    * length. Additionally, the benchmarks offer the following options: "-f"
    * can be used to report additional statistics of the benchmark, such as min
    * and max latencies and the number of iterations. "-m" option can be used
    * to set the maximum message length to be used in a benchmark. In the
    * default version, the benchmarks report the latencies for up to 1MB
    * message lengths. "-i" can be used to set the number of iterations to run
    * for each message length.

Point-to-Point UPC++ Benchmarks
-------------------------------
osu_upcxx_async_copy_put.c - Put Latency
    * This benchmark measures the latency of the UPC++ async_copy operation
    * between multiple UPC++ threads. In this benchmark, UPC+ threads with
    * ranks less than (THREADS/2) issues UPC++ async_copy from local to remote
    * memory on peer threads. Peer threads are identified as
    * (MYTHREAD+THREADS/2). This is repeated for a fixed number of iterations,
    * for varying data sizes. The average latency per iteration is reported. A
    * few warm-up iterations are run without timing to ignore any start-up
    * overheads. All UPC++ threads call barrier after the test for each message
    * size.

osu_upcxx_async_copy_get.c - Get Latency
    * This benchmark is similar as the osu_upcxx_async_copy_put benchmark that
    * is described above. The difference is that the async_copy operation
    * copies from remote to local memory. The average get operation latency per
    * iteration is reported.

Collective UPC++ Benchmarks
---------------------------
osu_upcxx_allgather - UPC++ Allgather Latency Test
osu_upcxx_alltoall  - UPC++ Alltoall Latency Test
osu_upcxx_bcast     - UPC++ Broadcast Latency Test
osu_upcxx_gather    - UPC++ Gather Latency Test
osu_upcxx_reduce    - UPC++ Reduce Latency Test
osu_upcxx_scatter   - UPC++ Scatter Latency Test

Collective Latency Tests
    * The latest OMB Version includes benchmarks for various UPC++ collective
    * operations (upcxx_allgather, upcxx_alltoall, upcxx_bcast, upcxx_gather,
    * upcxx_reduce, and upcxx_scatter).  These benchmarks work in the following
    * manner. Suppose users run the osu_upcxx_bcast benchmark with N processes,
    * the benchmark measures the min, max and the average latency of the
    * upcxx_bcast collective operation across N processes, for various message
    * lengths, over a large number of iterations. In the default version, these
    * benchmarks report the average latency for each message length.
    * Additionally, the benchmarks offer the following options:
    * "-f" can be used to report additional statistics of the benchmark, such
    * as min and max latencies and the number of iterations.
    * "-m" option can be used to set the maximum message length to be used in a
    * benchmark. In the default version, the benchmarks report the latencies
    * for up to 1MB message lengths.
    * "-i" can be used to set the number of iterations to run for each message
    * length.

Startup Benchmarks
------------------
osu_init.c - This benchmark measures the minimum, maximum, and average time
    * each process takes to complete MPI_Init.

osu_hello.c - This is a simple hello world program. Users can take advantage of
    * this to time it takes for all processes to execute MPI_Init +
    * MPI_Finalize.
    *
    * Example:
    * - time mpirun_rsh -np 2 -hostfile hostfile osu_hello

ROCm, CUDA and OpenACC Extensions to OMB
----------------------------------------
CUDA Extensions to OMB can be enable by configuring the benchmark suite with
--enable-cuda option as shown below.

    ./configure CC=/path/to/mpicc
                CXX=/path/to/mpicxx
                --enable-cuda
                --with-cuda-include=/path/to/cuda/include
                --with-cuda-libpath=/path/to/cuda/lib
    make
    make install

ROCm extensions can be enabled by configuring OMB with --enable-rocm
option as shown below. Similarly OpenACC extensions can be enabled using
--enable-openacc option. The MPI library used should be able to support MPI
communication from buffers in GPU Device memory.

    ./configure CC=/path/to/mpicc
                CXX=/path/to/mpicxx
                --enable-rocm
                --with-rocm=/path/to/rocm/install
    make
    make install

Similarly, OpenACC and SYCL Extensions can be enabled by specifying the
--enable-openacc and --enable-sycl options respectively.  The MPI library used
should be able to support MPI communication from buffers in GPU Device memory.

The following benchmarks have been extended to evaluate performance of
MPI communication using buffers on AMD and NVIDIA GPU devices.

    osu_bibw           - Bidirectional Bandwidth Test
    osu_bw             - Bandwidth Test
    osu_latency        - Latency Test
    osu_mbw_mr         - Multiple Bandwidth / Message Rate Test
    osu_multi_lat      - Multi-pair Latency Test
    osu_put_latency    - Latency Test for Put
    osu_get_latency    - Latency Test for Get
    osu_put_bw         - Bandwidth Test for Put
    osu_get_bw         - Bandwidth Test for Get
    osu_put_bibw       - Bidirectional Bandwidth Test for Put
    osu_acc_latency    - Latency Test for Accumulate
    osu_cas_latency    - Latency Test for Compare and Swap
    osu_fop_latency    - Latency Test for Fetch and Op
    osu_allgather      - MPI_Allgather Latency Test
    osu_allgatherv     - MPI_Allgatherv Latency Test
    osu_allreduce      - MPI_Allreduce Latency Test
    osu_alltoall       - MPI_Alltoall Latency Test
    osu_alltoallv      - MPI_Alltoallv Latency Test
    osu_bcast          - MPI_Bcast Latency Test
    osu_gather         - MPI_Gather Latency Test
    osu_gatherv        - MPI_Gatherv Latency Test
    osu_reduce         - MPI_Reduce Latency Test
    osu_reduce_scatter - MPI_Reduce_scatter Latency Test
    osu_scatter        - MPI_Scatter Latency Test
    osu_scatterv       - MPI_Scatterv Latency Test
    osu_iallgather     - MPI_Iallgather Latency Test
    osu_iallgatherv    - MPI_Iallgatherv Latency Test
    osu_iallreduce     - MPI_Iallreduce Latency Test
    osu_ialltoall      - MPI_Ialltoall Latency Test
    osu_ialltoallv     - MPI_Ialltoallv Latency Test
    osu_ialltoallw     - MPI_Ialltoallw Latency Test
    osu_ibcast         - MPI_Ibcast Latency Test
    osu_igather        - MPI_Igather Latency Test
    osu_igatherv       - MPI_Igatherv Latency Test
    osu_ireduce        - MPI_Ireduce Latency Test
    osu_iscatter       - MPI_Iscatter Latency Test
    osu_iscatterv      - MPI_Iscatterv Latency Test

If both CUDA and OpenACC support is enabled you can switch between the modes
using the -d [cuda|openacc] option to the benchmarks. If ROCm support is
enabled, you need to use -d rocm option to make the benchmarks use this feature.
If SYCL support is enabled, you need to use -d sycl option to make the
benchmarks use this feature. Whether a process allocates its communication
buffers on the GPU device or on the host can be controlled at run-time.
Use the -h option for more help.

    ./osu_latency -h
    Usage: osu_latency [options] [RANK0 RANK1]

    RANK0 and RANK1 may be `D' or `H' which specifies whether
    the buffer is allocated on the accelerator device or host
    memory for each mpi rank

    options:
      -d TYPE   accelerator device buffers can be of TYPE `cuda' or `openacc'
      -h        print this help message

Each of the pt2pt benchmarks takes two input parameters. The first parameter
indicates the location of the buffers at rank 0 and the second parameter
indicates the location of the buffers at rank 1. The value of each of these
parameters can be either 'H' or 'D' to indicate if the buffers are to be on the
host or on the device respectively. When no parameters are specified, the
buffers are allocated on the host.  The collective benchmarks will use buffers
allocated on the device if the -d option is used otherwise the buffers will be
allocated on the host.

Examples:

    - mpirun_rsh -np 2 -hostfile hostfile MV2_USE_CUDA=1 osu_latency D D
    - mpirun_rsh -np 2 -hostfile hostfile MV2_USE_ROCM=1 osu_latency D D

In this run, the latency test allocates buffers at both rank 0 and rank 1 on
the GPU devices.

    - mpirun_rsh -np 2 -hostfile hostfile MV2_USE_CUDA=1 osu_bw D H
    - mpirun_rsh -np 2 -hostfile hostfile MV2_USE_ROCM=1 osu_bw D H

In this run, the bandwidth test allocates buffers at rank 0 on the GPU device
and buffers at rank 1 on the host.

Setting GPU affinity
--------------------
GPU affinity for processes is set before MPI_Init is called in the benchmarks.
The process rank on a node is normally used to do this and different MPI
launchers expose this information through different environment variables. The
benchmarks use an environment variable called LOCAL_RANK to get this
information.

Starting with OMB v5.4.4, the benchmarks automatically identify the process rank
on a node for MVAPICH2 when launched with mpirun_rsh. However, a script like
below can be used to export this environment variable when using OMB to work
with other MPI launchers and libraries.

    #!/bin/bash

    export LOCAL_RANK=$MV2_COMM_WORLD_LOCAL_RANK
    exec $*

A copy of this script is installed as get_local_rank alongside the benchmarks.
It can be used as follows:

    mpirun_rsh -np 2 -hostfile hostfile MV2_USE_CUDA=1 get_local_rank \
        ./osu_latency D D
    mpirun_rsh -np 2 -hostfile hostfile MV2_USE_ROCM=1 get_local_rank \
        ./osu_latency D D

Support for Derived Data Types
------------------------------
The following benchmarks have been extended to support MPI derived data types in
addition to the communication performance.

    osu_bibw           - Bidirectional Bandwidth Test
    osu_bw             - Bandwidth Test
    osu_latency        - Latency Test
    osu_mbw_mr         - Multiple Bandwidth / Message Rate Test
    osu_multi_lat      - Multi-pair Latency Test
    osu_latency_mt     - Multi-threaded Latency Test
    osu_latency_mp     - Multi-process Latency Test
    osu_allgather      - MPI_Allgather Latency Test(*)
    osu_allgatherv     - MPI_Allgatherv Latency Test
    osu_alltoall       - MPI_Alltoall Latency Test
    osu_alltoallv      - MPI_Alltoallv Latency Test
    osu_bcast          - MPI_Bcast Latency Test
    osu_gather         - MPI_Gather Latency Test(*)
    osu_gatherv        - MPI_Gatherv Latency Test
    osu_scatter        - MPI_Scatter Latency Test(*)
    osu_scatterv       - MPI_Scatterv Latency Test
    osu_iallgather     - MPI_Iallgather Latency Test
    osu_iallgatherv    - MPI_Iallgatherv Latency Test
    osu_ialltoall      - MPI_Ialltoall Latency Test
    osu_ialltoallv     - MPI_Ialltoallv Latency Test
    osu_ialltoallw     - MPI_Ialltoallw Latency Test
    osu_ibcast         - MPI_Ibcast Latency Test
    osu_igather        - MPI_Igather Latency Test
    osu_igatherv       - MPI_Igatherv Latency Test
    osu_iscatter       - MPI_Iscatter Latency Test
    osu_iscatterv      - MPI_Iscatterv Latency Test

    * Additionally, the benchmarks offer following options:
    * "-D" option can be used to enable DDT support
    *      "-D cont" for contiguous DDT support
    *      "-D vect:[stride]:[block_length]" for vector DDT support
    *           Example: -D vect:8:4
    *      "-D indx:[ddt file path]" for index DDT support
    *           Example: -D indx:ddt_sample.txt

Support for Graphs
------------------
The following benchmarks have been extended to support graphing of metrics in
addition to the communication performance.

This feature can be used if 'gnuplot' in present in the PATH or by passing gnuplot
installation path during configure time using
--with-gnuplot=/path/to/gnuplot/installation/bin.

For graph output in pdf format, in addition to gnuplot, 'convert' must be present
in the PATH or must be passed during configure time using
--with-convert=/path/to/ImageMagick/installation/bin

    osu_bibw           - Bidirectional Bandwidth Test
    osu_bw             - Bandwidth Test
    osu_latency        - Latency Test
    osu_mbw_mr         - Multiple Bandwidth / Message Rate Test
    osu_multi_lat      - Multi-pair Latency Test
    osu_latency_mt     - Multi-threaded Latency Test
    osu_latency_mp     - Multi-process Latency Test
    osu_allgather      - MPI_Allgather Latency Test
    osu_allgatherv     - MPI_Allgatherv Latency Test
    osu_alltoall       - MPI_Alltoall Latency Test
    osu_alltoallv      - MPI_Alltoallv Latency Test
    osu_alltoallw      - MPI_Alltoallw Latency Test
    osu_allreduce      - MPI_Allreduce Latency Test
    osu_bcast          - MPI_Bcast Latency Test
    osu_gather         - MPI_Gather Latency Test
    osu_gatherv        - MPI_Gatherv Latency Test
    osu_barrier        - MPI_Barrier Latency Test
    osu_reduce         - MPI_Reduce Latency Test
    osu_reduce_scatter - MPI_Reduce_scatter Latency Test
    osu_scatter        - MPI_Scatter Latency Test
    osu_scatterv       - MPI_Scatterv Latency Test
    osu_iallgather     - MPI_Iallgather Latency Test
    osu_iallgatherv    - MPI_Iallgatherv Latency Test
    osu_iallreduce     - MPI_Iallreduce Latency Test
    osu_ialltoall      - MPI_Ialltoall Latency Test
    osu_ialltoallv     - MPI_Ialltoallv Latency Test
    osu_ialltoallw     - MPI_Ialltoallw Latency Test
    osu_ibarrier       - MPI_Ibarrier Latency Test
    osu_ibcast         - MPI_Ibcast Latency Test
    osu_igather        - MPI_Igather Latency Test
    osu_igatherv       - MPI_Igatherv Latency Test
    osu_ireduce        - MPI_Ireduce Latency Test
    osu_iscatter       - MPI_Iscatter Latency Test
    osu_iscatterv      - MPI_Iscatterv Latency Test
    osu_put_latency    - Latency Test for Put
    osu_get_latency    - Latency Test for Get
    osu_put_bw         - Bandwidth Test for Put
    osu_get_bw         - Bandwidth Test for Get
    osu_put_bibw       - Bidirectional Bandwidth Test for Put
    osu_acc_latency    - Latency Test for Accumulate
    osu_cas_latency    - Latency Test for Compare and Swap
    osu_fop_latency    - Latency Test for Fetch and Op
    osu_get_acc_latency- Latency Test for Get_accumulate with Active/Passive

    * Additionally, the benchmarks offer following options:
    * "-G" option can be used to output result in graphs
    *      "-G tty" for graph output in terminal using ASCII characters
    *      "-G png" for graph output in png format
    *      "-G pdf" for graph output in pdf format

Support for PAPI
------------------
The following benchmarks have been extended to support PAPI to get insights in
addition to the communication performance.

This feature can be used by enabling PAPI support using --enable-papi and
passing PAPI installation path during configure time using
--with-papi=/path/to/papi/build.

    osu_bibw           - Bidirectional Bandwidth Test
    osu_bw             - Bandwidth Test
    osu_latency        - Latency Test
    osu_mbw_mr         - Multiple Bandwidth / Message Rate Test
    osu_multi_lat      - Multi-pair Latency Test
    osu_latency_mp     - Multi-process Latency Test
    osu_allgather      - MPI_Allgather Latency Test
    osu_allgatherv     - MPI_Allgatherv Latency Test
    osu_alltoall       - MPI_Alltoall Latency Test
    osu_alltoallv      - MPI_Alltoallv Latency Test
    osu_alltoallw      - MPI_Alltoallw Latency Test
    osu_allreduce      - MPI_Allreduce Latency Test
    osu_bcast          - MPI_Bcast Latency Test
    osu_gather         - MPI_Gather Latency Test
    osu_gatherv        - MPI_Gatherv Latency Test
    osu_barrier        - MPI_Barrier Latency Test
    osu_reduce         - MPI_Reduce Latency Test
    osu_reduce_scatter - MPI_Reduce_scatter Latency Test
    osu_ireduce_scatter- MPI_Ireduce_scatter Latency Test
    osu_scatter        - MPI_Scatter Latency Test
    osu_scatterv       - MPI_Scatterv Latency Test
    osu_iallgather     - MPI_Iallgather Latency Test
    osu_iallgatherv    - MPI_Iallgatherv Latency Test
    osu_iallreduce     - MPI_Iallreduce Latency Test
    osu_ialltoall      - MPI_Ialltoall Latency Test
    osu_ialltoallv     - MPI_Ialltoallv Latency Test
    osu_ialltoallw     - MPI_Ialltoallw Latency Test
    osu_ibarrier       - MPI_Ibarrier Latency Test
    osu_ibcast         - MPI_Ibcast Latency Test
    osu_igather        - MPI_Igather Latency Test
    osu_igatherv       - MPI_Igatherv Latency Test
    osu_ireduce        - MPI_Ireduce Latency Test
    osu_iscatter       - MPI_Iscatter Latency Test
    osu_iscatterv      - MPI_Iscatterv Latency Test
    osu_put_latency    - Latency Test for Put
    osu_get_latency    - Latency Test for Get
    osu_put_bw         - Bandwidth Test for Put
    osu_get_bw         - Bandwidth Test for Get
    osu_put_bibw       - Bidirectional Bandwidth Test for Put
    osu_acc_latency    - Latency Test for Accumulate
    osu_cas_latency    - Latency Test for Compare and Swap
    osu_fop_latency    - Latency Test for Fetch and Op
    osu_get_acc_latency- Latency Test for Get_accumulate with Active/Passive


    * Additionally, the benchmarks offer following options:
    * "-P  [EVENTS]:[PATH]" option can be used to enable PAPI output
    *      "[EVENTS]" Comma seperated list of PAPI events
    *      "[PATH]" PAPI output file path
    * E.g: -PPAPI_L1_DCM,PAPI_VEC_DP,PAPI_VEC_SP:papi_output.out

Point-to-Point Persistent Benchmarks
------------------------------
osu_latency_p - Persistent Latency Test
    * Similar to osu_latency test using persistent point-to-point communication
    * requests.

osu_bw_p - Persistent Bandwidth Test
    * Similar to osu_bw test using persistent point-to-point communication
    * requests.

osu_bibw_p - Persistent Bidirectional Bandwidth Test
    * Similar to osu_bibw test using persistent point-to-point communication
    * requests.

Neighborhood Collective MPI Benchmarks
--------------------------------------
osu_neighbor_allgather  - MPI_Neighbor_allgather Latency Test
osu_neighbor_allgatherv - MPI_Neighbor_allgatherv Latency Test
osu_neighbor_alltoall   - MPI_Neighbor_alltoall Latency Test
osu_neighbor_alltoallv  - MPI_Neighbor_alltoallv Latency Test
osu_neighbor_alltoallw  - MPI_Neighbor_alltoallw Latency Test
osu_ineighbor_allgather - MPI_Ineighbor_allgather Latency Test
osu_ineighbor_allgatherv- MPI_Ineighbor_allgatherv Latency Test
osu_ineighbor_alltoall  - MPI_Ineighbor_alltoall Latency Test
osu_ineighbor_alltoallv - MPI_Ineighbor_alltoallv Latency Test
osu_ineighbor_alltoallw - MPI_Ineighbor_alltoallw Latency Test

Neighbor Collective Latency Tests
    * In addition to the blocking and non-blocking collective latency tests,
    * we provide several neighborhood collectives as mentioned above.
    * Additionally, the benchmarks offer following options:
    * "-N [TYPE]:[ARGS]  Configure neighborhood collectives.(default:- cart:1:1)
    *                    -N cart:<num of dimentions:radius>   //Cartesian
    *                    -N graph:<adjacency graph file>      //Graph
    * Sample adjacency graph file found in utils.

Support for multiple MPI types
--------------------------------------
The following benchmarks have been extended to support multiple MPI types.
    osu_bibw                - Bidirectional Bandwidth Test
    osu_bw                  - Bandwidth Test
    osu_latency             - Latency Test
    osu_mbw_mr              - Multiple Bandwidth / Message Rate Test
    osu_multi_lat           - Multi-pair Latency Test
    osu_latency_mp          - Multi-process Latency Test
    osu_allgather           - MPI_Allgather Latency Test
    osu_allgatherv          - MPI_Allgatherv Latency Test
    osu_alltoall            - MPI_Alltoall Latency Test
    osu_alltoallv           - MPI_Alltoallv Latency Test
    osu_alltoallw           - MPI_Alltoallw Latency Test
    osu_allreduce           - MPI_Allreduce Latency Test
    osu_bcast               - MPI_Bcast Latency Test
    osu_gather              - MPI_Gather Latency Test
    osu_gatherv             - MPI_Gatherv Latency Test
    osu_barrier             - MPI_Barrier Latency Test
    osu_reduce              - MPI_Reduce Latency Test
    osu_reduce_scatter      - MPI_Reduce_scatter Latency Test
    osu_ireduce_scatter     - MPI_Ireduce_scatter Latency Test
    osu_scatter             - MPI_Scatter Latency Test
    osu_scatterv            - MPI_Scatterv Latency Test
    osu_iallgather          - MPI_Iallgather Latency Test
    osu_iallgatherv         - MPI_Iallgatherv Latency Test
    osu_iallreduce          - MPI_Iallreduce Latency Test
    osu_ialltoall           - MPI_Ialltoall Latency Test
    osu_ialltoallv          - MPI_Ialltoallv Latency Test
    osu_ialltoallw          - MPI_Ialltoallw Latency Test
    osu_ibarrier            - MPI_Ibarrier Latency Test
    osu_ibcast              - MPI_Ibcast Latency Test
    osu_igather             - MPI_Igather Latency Test
    osu_igatherv            - MPI_Igatherv Latency Test
    osu_ireduce             - MPI_Ireduce Latency Test
    osu_iscatter            - MPI_Iscatter Latency Test
    osu_iscatterv           - MPI_Iscatterv Latency Test
    osu_put_latency         - Latency Test for Put
    osu_get_latency         - Latency Test for Get
    osu_put_bw              - Bandwidth Test for Put
    osu_get_bw              - Bandwidth Test for Get
    osu_put_bibw            - Bidirectional Bandwidth Test for Put
    osu_acc_latency         - Latency Test for Accumulate
    osu_cas_latency         - Latency Test for Compare and Swap
    osu_fop_latency         - Latency Test for Fetch and Op
    osu_get_acc_latency     - Latency Test for Get_accumulate with Active/Passive
    osu_neighbor_allgather  - MPI_Neighbor_allgather Latency Test
    osu_neighbor_allgatherv - MPI_Neighbor_allgatherv Latency Test
    osu_neighbor_alltoall   - MPI_Neighbor_alltoall Latency Test
    osu_neighbor_alltoallv  - MPI_Neighbor_alltoallv Latency Test
    osu_neighbor_alltoallw  - MPI_Neighbor_alltoallw Latency Test
    osu_ineighbor_allgather - MPI_Ineighbor_allgather Latency Test
    osu_ineighbor_allgatherv- MPI_Ineighbor_allgatherv Latency Test
    osu_ineighbor_alltoall  - MPI_Ineighbor_alltoall Latency Test
    osu_ineighbor_alltoallv - MPI_Ineighbor_alltoallv Latency Test
    osu_ineighbor_alltoallw - MPI_Ineighbor_alltoallw Latency Test

    * Additionally, the benchmarks offer following options:
    * "-T" option can be used to pass required MPI type.
    *      "-T mpi_int" for running benchmark using MPI_INT.
    *      "-T all" for running benchmark using supported MPI types.

Support for MPI Session
--------------------------------------
The following benchmarks have been extended to support MPI initialization using
session.
    osu_bibw                - Bidirectional Bandwidth Test
    osu_bw                  - Bandwidth Test
    osu_latency             - Latency Test
    osu_mbw_mr              - Multiple Bandwidth / Message Rate Test
    osu_multi_lat           - Multi-pair Latency Test
    osu_latency_mp          - Multi-process Latency Test
    osu_latency_mt          - Multi-threaded Latency Test
    osu_latency_p           - Persistent Latency Test
    osu_bw_p                - Persistent Bandwidth Test
    osu_bibw_p              - Persistent Bidirectional Bandwidth Test
    osu_allgather           - MPI_Allgather Latency Test
    osu_allgatherv          - MPI_Allgatherv Latency Test
    osu_alltoall            - MPI_Alltoall Latency Test
    osu_alltoallv           - MPI_Alltoallv Latency Test
    osu_alltoallw           - MPI_Alltoallw Latency Test
    osu_allreduce           - MPI_Allreduce Latency Test
    osu_bcast               - MPI_Bcast Latency Test
    osu_gather              - MPI_Gather Latency Test
    osu_gatherv             - MPI_Gatherv Latency Test
    osu_barrier             - MPI_Barrier Latency Test
    osu_reduce              - MPI_Reduce Latency Test
    osu_reduce_scatter      - MPI_Reduce_scatter Latency Test
    osu_ireduce_scatter     - MPI_Ireduce_scatter Latency Test
    osu_scatter             - MPI_Scatter Latency Test
    osu_scatterv            - MPI_Scatterv Latency Test
    osu_iallgather          - MPI_Iallgather Latency Test
    osu_iallgatherv         - MPI_Iallgatherv Latency Test
    osu_iallreduce          - MPI_Iallreduce Latency Test
    osu_ialltoall           - MPI_Ialltoall Latency Test
    osu_ialltoallv          - MPI_Ialltoallv Latency Test
    osu_ialltoallw          - MPI_Ialltoallw Latency Test
    osu_ibarrier            - MPI_Ibarrier Latency Test
    osu_ibcast              - MPI_Ibcast Latency Test
    osu_igather             - MPI_Igather Latency Test
    osu_igatherv            - MPI_Igatherv Latency Test
    osu_ireduce             - MPI_Ireduce Latency Test
    osu_iscatter            - MPI_Iscatter Latency Test
    osu_iscatterv           - MPI_Iscatterv Latency Test
    osu_neighbor_allgather  - MPI_Neighbor_allgather Latency Test
    osu_neighbor_allgatherv - MPI_Neighbor_allgatherv Latency Test
    osu_neighbor_alltoall   - MPI_Neighbor_alltoall Latency Test
    osu_neighbor_alltoallv  - MPI_Neighbor_alltoallv Latency Test
    osu_neighbor_alltoallw  - MPI_Neighbor_alltoallw Latency Test
    osu_ineighbor_allgather - MPI_Ineighbor_allgather Latency Test
    osu_ineighbor_allgatherv- MPI_Ineighbor_allgatherv Latency Test
    osu_ineighbor_alltoall  - MPI_Ineighbor_alltoall Latency Test
    osu_ineighbor_alltoallv - MPI_Ineighbor_alltoallv Latency Test
    osu_ineighbor_alltoallw - MPI_Ineighbor_alltoallw Latency Test
    osu_put_latency         - Latency Test for Put
    osu_get_latency         - Latency Test for Get
    osu_put_bw              - Bandwidth Test for Put
    osu_get_bw              - Bandwidth Test for Get
    osu_put_bibw            - Bidirectional Bandwidth Test for Put
    osu_acc_latency         - Latency Test for Accumulate
    osu_cas_latency         - Latency Test for Compare and Swap
    osu_fop_latency         - Latency Test for Fetch and Op
    osu_get_acc_latency     - Latency Test for Get_accumulate with Active/Passive
    osu_init                - MPI initialization test

    * Additionally, the benchmarks offer following options:
    * "-I" option can be used to enable MPI initialization using session.

Support for MPI_IN_PLACE
--------------------------------------
The following benchmarks have been extended to support MPI_IN_PLACE.
    osu_allgather           - MPI_Allgather Latency Test
    osu_allgatherv          - MPI_Allgatherv Latency Test
    osu_alltoall            - MPI_Alltoall Latency Test
    osu_alltoallv           - MPI_Alltoallv Latency Test
    osu_alltoallw           - MPI_Alltoallw Latency Test
    osu_allreduce           - MPI_Allreduce Latency Test
    osu_gather              - MPI_Gather Latency Test
    osu_gatherv             - MPI_Gatherv Latency Test
    osu_reduce              - MPI_Reduce Latency Test
    osu_reduce_scatter      - MPI_Reduce_scatter Latency Test
    osu_ireduce_scatter     - MPI_Ireduce_scatter Latency Test
    osu_scatter             - MPI_Scatter Latency Test
    osu_scatterv            - MPI_Scatterv Latency Test
    osu_iallgather          - MPI_Iallgather Latency Test
    osu_iallgatherv         - MPI_Iallgatherv Latency Test
    osu_iallreduce          - MPI_Iallreduce Latency Test
    osu_ialltoall           - MPI_Ialltoall Latency Test
    osu_ialltoallv          - MPI_Ialltoallv Latency Test
    osu_ialltoallw          - MPI_Ialltoallw Latency Test
    osu_igather             - MPI_Igather Latency Test
    osu_igatherv            - MPI_Igatherv Latency Test
    osu_ireduce             - MPI_Ireduce Latency Test
    osu_iscatter            - MPI_Iscatter Latency Test
    osu_iscatterv           - MPI_Iscatterv Latency Test

    * Additionally, the benchmarks offer following options:
    * "-l" option can be used to run benchmarks with MPI_IN_PLACE support.

Option to set root rank
--------------------------------------
The following benchmarks have been extended which an option to set root rank.
    osu_gather              - MPI_Gather Latency Test
    osu_gatherv             - MPI_Gatherv Latency Test
    osu_reduce              - MPI_Reduce Latency Test
    osu_scatter             - MPI_Scatter Latency Test
    osu_scatterv            - MPI_Scatterv Latency Test
    osu_igather             - MPI_Igather Latency Test
    osu_igatherv            - MPI_Igatherv Latency Test
    osu_ireduce             - MPI_Ireduce Latency Test
    osu_iscatter            - MPI_Iscatter Latency Test
    osu_iscatterv           - MPI_Iscatterv Latency Test

    * Additionally, the benchmarks offer following options:
    * "-r" option can be used to set the root rank of the collective.
    *      "-r rotate" to change root for every iteration.
    *      "-r fixed:<rank>" to run benchmark with a fixed root.

Option to print tail latencies/bandwidth
-----------------------------------------
Benchmarks have been extended to support the following additional metrics by
default,
    * 50th percentile tail latency/bandwidth
    * 95th percentile tail latency/bandwidth
    * 99th percentile tail latency/bandwidth

The benchmarks offer following options:
    * "-z" Outputs P99, P90, P50 percentiles"
    * "-z<1-99,1-99,1-99..>" Comma seperated percentile range

Building with MPI-4 support
--------------------------------------
OMB supports some of the MPI-4 features like MPI Sessions and Persistenct
Collectives. When an MPI-4 supported MPI library is used at the time of
configuration, OMB will automatically detect and build with MPI-4 support.

OMB can also be forced to build with MPI-4 support by passing "--enable-mpi4"
during configure time.


Congestion Benchmarks
-----------------------------------
osu_bw_fan_out.c - benchmark to measure send bandwidth.
    * The fan out congestion bandwidth test evaluates the aggregate
    * uni-directional bandwidth between multiple pairs of process across nodes.
    * Each of the sending processes on one node sends a fixed number of
    * messages (the window size) back-to-back to the paired receiving
    * processies on k nodes before waiting for a reply from the receiver.
    * This process is repeated for several iterations. The objective of this
    * benchmark is to determine the achieved bandwidth with configurable
    * number of processes running on each node.

osu_bw_fan_in.c - benchmark to measure receive bandwidth.
    * The fan in congestion bandwidth test evaluates the aggregate
    * uni-directional bandwidth between multiple pairs of process across nodes.
    * Each of the sending processes on k nodes sends a fixed number of
    * messages (the window size) back-to-back to the paired receiving
    * processies on 1 node before waiting for a reply from the receiver.
    * This process is repeated for several iterations. The objective of this
    * benchmark is to determine the achieved bandwidth with configurable
    * number of processes running on each node.

Point-to-Point Partitioned Benchmarks
------------------------------
osu_partitioned_latency - Partitioned Latency Test
    * Similar to osu_latency test using partitioned point-to-point communication
    * requests.
