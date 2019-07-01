# warc-compression
Scripts to experiment with different compression choices for WARCs. In particular:

- gzip, from level 1 to 9
- lz4, from level 1 to 9
- zstd, from level 1 to 19
- zstd with dictionary, from level 1 to 19

The variables of interest are:

- how long does it take to compress?
- how well does it compress?
- how long does it take to uncompress?

For our purposes, we'd like to only measure the CPU - not network I/O
nor disk I/O.

## Benchmarking

Benchmarking on an r5a.xlarge works well. It has 32GB of RAM, of which we can
allocate 24GB to a RAM drive so that we're not benchmarking fs performance.

```
# Make a ramdisk so we're not accidentally benchmarking EBS
sudo mkdir -p /mnt/ramdisk
sudo mount -t tmpfs -o size=24G tmpfs /mnt/ramdisk

# Install supporting tools
sudo apt-get update
sudo apt-get install liblz4-tool build-essential

git clone https://github.com/facebook/zstd.git
cd zstd
make -j4
sudo make install

# Fetch sample files
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2019-22/segments/1558232255092.55/warc/CC-MAIN-20190519181530-20190519203530-00051.warc.gz -O train.warc.gz
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2019-22/segments/1558232255092.55/wet/CC-MAIN-20190519181530-20190519203530-00051.warc.wet.gz -O train.wet.gz
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2019-22/segments/1558232255092.55/wat/CC-MAIN-20190519181530-20190519203530-00051.warc.wat.gz -O train.wat.gz

wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/warc/CC-MAIN-20181212065445-20181212090945-00451.warc.gz -O test.warc.gz
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/wet/CC-MAIN-20181212065445-20181212090945-00451.warc.wet.gz -O test.wet.gz
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/wat/CC-MAIN-20181212065445-20181212090945-00451.warc.wat.gz -O test.wat.gz
```
