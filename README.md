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

git clone https://github.com/cldellow/warc-compression.git
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

# Benchmarking generic dict vs custom dict
```
~/warc-compression/extract warc 
cp -ar e e.bak
cp -ar s s.bak

# Create a dictionary trained on everything
zstd -9 -o dict.generic --train s/*

# Create train/test folders for metadata and requests
mkdir sm em sr er
cp $(grep -l WARC-Type:.metadata s/*) sm
cp $(grep -l WARC-Type:.metadata e/*) em
cp $(grep -l WARC-Type:.request s/*) sr
cp $(grep -l WARC-Type:.request e/*) er

# Measure raw sizes
du --bytes er em
#34893842        er
#32600931        em

# Compress with zstd -9 as a baseline
zstd -9 er/* em/*
for x in em er; do echo -n "$x "; ls -l $x/*.zst | awk '{ N += $5 } END { print N}'; done
#em 22478177
#er 22555102

# Compress with generic dictionary
rm er/*.zst em/*.zst
zstd -9 -D dict.generic er/* em/*
for x in em er; do echo -n "$x "; ls -l $x/*.zst | awk '{ N += $5 } END { print N}'; done
#em 9789227
#er 9130685
rm er/*.zst em/*.zst

# Create specific dictionaries, compress with them
zstd -9 -o dict.metadata --train sm/*
zstd -9 -o dict.request --train sr/*
zstd -9 -D dict.request er/*
zstd -9 -D dict.metadata em/*
for x in em er; do echo -n "$x "; ls -l $x/*.zst | awk '{ N += $5 } END { print N}'; done
#em 8488818
#er 8183580

# Filter responses that are a specific language. This requires a bit of a process:
# 1) Copy all responses to own folder
# 2) Copy all requests that have cld2 metadata for language X to own folder
# 3) Extract WARC-Concurrent-To fields from (2) to file
# 4) Use IDs in (3) to do fgrep in files in (1) for responsive records
```
