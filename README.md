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
