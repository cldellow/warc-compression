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

For our purposes, we'd like to only measure the CPU

## Sample file

`example-warc.gz` is the result of `head -c1438655 CC-MAIN-20181113000225-20181113022225-00331.warc.gz`

Some sample files:

Training:

- http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2019-22/segments/1558232255092.55/warc/CC-MAIN-20190519181530-20190519203530-00051.warc.gz

Testing:

- http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/warc/CC-MAIN-20181212065445-20181212090945-00451.warc.gz

## Usage

```
./extract-entries /mnt/storage/cc/CC-MAIN-20181113000225-20181113022225-00331.warc.gz 1000
for x in {1..9}; do time gzip --to-stdout -"$x" entries/* | wc --bytes; done

for x in {1..19}; do time zstd --compress --stdout -"$x" entries/* | wc --bytes; done

# I don't think brotli supports the idea of "catable" streams.
# for x in {0..9}; do time brotli -"$x" --stdout --force entries/* | wc --bytes; done

# lz4 CLI tools don't support emitting one stream for multiple files;
# they'll write 1 file per input.
# for x in {1..9}; do time lz4 -"$x" --multiple entries/*; done
```

## Benchmarking

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

wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/warc/CC-MAIN-20181212065445-20181212090945-00451.warc.gz -O test.warc.gz
wget http://commoncrawl.s3.amazonaws.com/crawl-data/CC-MAIN-2018-51/segments/1544376823785.24/wet/CC-MAIN-20181212065445-20181212090945-00451.warc.wet.gz -O test.wet.gz
```
