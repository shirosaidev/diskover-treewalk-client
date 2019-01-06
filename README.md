# diskover-treewalk-client

The tree walk client can run directly on your storage server using Python 2.6+ (no other dependencies).

### Usage

First copy these two files, diskover-treewalk-client.py and scandir.py, to your storage server.

Next, on your diskoiver linux host (not on storage server) start diskover in socket server mode (proxy) for the tree walk client to communicate with and send batches of directory listings (pickle).

`python diskover.py -i diskover-indexname -d /mnt/isilon -a -L --debug`

And now to start the tree walk client on your storage

### CLI options

```
Usage: diskover-treewalk-client.py [options]

Options:
  --version             show program's version number and exit
  -h, --help            show this help message and exit
  -p HOST, --proxyhost=HOST
                        Hostname or IP of diskover proxy socket server
  -P PORT, --port=PORT  Port for diskover proxy socket server (default: 9998)
  -b BATCH_SIZE, --batchsize=BATCH_SIZE
                        Batchsize (num of directories) to send to diskover
                        proxy (default: 50)
  -n NUM_CONNECTIONS, --numconn=NUM_CONNECTIONS
                        Number of tcp connections to use (default: 5)
  -t TREEWALK_METHOD, --twmethod=TREEWALK_METHOD
                        Tree walk method to use. Options are: oswalk, scandir,
                        pscandir (parallel scandir), ls,
                        pls (parallel ls), metaspider (default: pscandir)
  -g PATHTOGLS, --pathtogls=PATHTOGLS
                        Path to GNU ls command for ls and lsthreaded (default:
                        /usr/local/bin/gls)
  -r ROOTDIR_LOCAL, --rootdirlocal=ROOTDIR_LOCAL
                        Local path on storage to crawl from
  -R ROOTDIR_REMOTE, --rootdirremote=ROOTDIR_REMOTE
                        Mount point directory for diskover and bots that is
                        same location as rootdirlocal
  -T NUM_SCANDIR_THREADS, --pscandirthreads=NUM_SCANDIR_THREADS
                        Number of threads for pscandir treewalk method
                        (default: cpu core count x 2)
  -s NUM_SPIDERS, --metaspiderthreads=NUM_SPIDERS
                        Number of threads for metaspider treewalk method
                        (default: cpu core count x 2)
  -l NUM_LS_THREADS, --lsthreads=NUM_LS_THREADS
                        Number of threads for pls treewalk method (default:
                        cpu core count x 2)
  -e EXCLUDED_DIR, --excludeddir=EXCLUDED_DIR
                        Additional directory to exclude, can use wildcard such
                        as dirname* or *dirname*
                        (default: .*,.snapshot,.Snapshot,.zfs)
```

### Example

`python diskover-treewalk-client.py -p 192.168.2.3 -t pscandir -r /ifs/data -R /mnt/isilon`

**host** and **port** are the ip or hostname and port (default 9998) of the diskover socket server.

**num_connections** is the number of threads to open for sending tcp streams to diskover socket server. Make sure to open as many or more socket connections in diskover.cfg in socket section for maxconnections.

**treewalk_method** can be one of **oswalk**, **scandir**, **pscandir**, **ls**, **pls**, or **metaspider**. pscandir (parallel scandir) is default and uses multi-threaded scandir. Metaspider method will collect all meta data locally and send it over to diskover proxy which then sends it to the bots to process. This eliminates the bots from doing meta stat calls over nfs/cifs which could help to index very large storage systems. Spider threads are adjustable by setting --metaspiderthreads NUM_SPIDERS.

**ls and pls (parallel ls) require GNU ls which you can set the location with -g**. If you are using Isilon storage and want to use ls/pls, you will need to copy gls binary in the `treewalk_client/freebsd10_gnu_ls` directory. gls should go to into /usr/local/bin. You will need to also create a symlink for libintl.so.8 `ln -s /usr/local/lib/libintl.so.9 /usr/local/lib/libintl.so.8` which is required by gls.

**rootdir_local** is the local directory path on the storage that you want to tree walk from.
**rootdir_remote** is the mount that the crawl bots see to the same directory on storage as **rootdir_local**, this path replaces **rootdir_local** by the client before sending to diskover proxy so bots can locate that path on their end. Set this to the same path (-d path) as used when starting diskover.py proxy.

**excluded_dir** are directories you want to exclude from sending to diskover proxy. You can add additional directories using --excludeddir dirname, example -e .backups -e .Backups .

Experiment with the batch size to get the best queue fill rate in Redis and best optimization of the tcp connection. The client sends up to num_connections simultaneous tcp streams of pickle data to the diskover proxy.

Using less than 50 batch size (number of directories), depending on how many files are in each directory, can have a negative impact on performance since more packets with less data are being sent. If most of your directories have tons of files in them, setting batch size to lower could help to keep bots busier and the queue more full.
