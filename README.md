HBaseDS  [![Java CI with Maven](https://github.com/appform-io/HBaseDS/actions/workflows/maven.yml/badge.svg)](https://github.com/appform-io/HBaseDS/actions/workflows/maven.yml)
------------
Source project - https://github.com/sematext/HBaseWD

**The source library seems unmaintained. Hence following is the documentation inspired by the source which I will maintain. Contributions are appreciated. **

Description:
------------

HBaseWD stands for Distributing(sequential) Writes.

* writing records with sequential row keys (e.g. time-series data with row key built based on ts)
* using random unique IDs for records

First approach makes possible to perform fast range scans with help of setting start/stop keys on Scanner, but creates single region server hot-spotting problem upon writing data (as row keys go in sequence all records end up written into a single region at a time).

Second approach aims for fastest writing performance by distributing new records over random regions but makes not possible doing fast range scans against written data.

The suggested approach stays in the middle of the two above and proved to perform well by distributing records over the cluster during writing data while allowing range scans over it. HBaseWD provides very simple API to work with which makes it perfect to use with existing code.

Please refer to unit-tests for lib usage info as they aimed to act as example.

Blog:
-----
http://blog.sematext.com/2012/04/09/hbasewd-avoid-regionserver-hotspotting-despite-writing-records-with-sequential-keys/

Usage:
------
Distributing records with sequential keys which are being written in up to Byte.MAX_VALUE buckets:

    byte bucketsCount = (byte) 32; // distributing into 32 buckets
    RowKeyDistributor keyDistributor = new RowKeyDistributorByOneBytePrefix(bucketsCount);
    for (int i = 0; i < 100; i++) {
      Put put = new Put(keyDistributor.getDistributedKey(originalKey));
      ... // add values
      hTable.put(put);
    }


Performing a range scan over written data (internally <bucketsCount> scanners executed):

    Scan scan = new Scan(startKey, stopKey);
    ResultScanner rs = DistributedScanner.create(hTable, scan, keyDistributor);
    for (Result current : rs) {
      ...
    }

Performing mapreduce job over written data chunk specified by Scan:

    Configuration conf = HBaseConfiguration.create();
    Job job = new Job(conf, "testMapreduceJob");

    Scan scan = new Scan(startKey, stopKey);

    TableMapReduceUtil.initTableMapperJob("table", scan, RowCounterMapper.class, ImmutableBytesWritable.class, Result.class, job);

    // Substituting standard TableInputFormat which was set in
    // TableMapReduceUtil.initTableMapperJob(...)

    job.setInputFormatClass(WdTableInputFormat.class);
    keyDistributor.addInfo(job.getConfiguration());

Another useful RowKeyDistributor is RowKeyDistributorByHashPrefix. Please see
example below. It will creates "distributed key" based on original key value
so that later when you have original key and want to update the record you can
calculate distributed key without roundtrip to HBase.

    AbstractRowKeyDistributor keyDistributor = new RowKeyDistributorByHashPrefix(new RowKeyDistributorByHashPrefix.OneByteSimpleHash(15));

You can use your own hashing logic here by implementing simple interface:

    public static interface Hasher extends Parametrizable {
      byte[] getHashPrefix(byte[] originalKey);
      byte[][] getAllPossiblePrefixes();
    }



Extending Row Keys Distributing Patterns:
------------

HBaseWD is designed to be flexible and to support custom row key distribution approaches. To define custom row key distributing logic just implement _AbstractRowKeyDistributor_ abstract class which is really very simple:

    public abstract class AbstractRowKeyDistributor implements Parametrizable {
      public abstract byte[] getDistributedKey(byte[] originalKey);
      public abstract byte[] getOriginalKey(byte[] adjustedKey);
      public abstract byte[][] getAllDistributedKeys(byte[] originalKey);
      ... // some utility methods
    }

Changes
------------
The libraries used (hbase, hadoop etc) has been upgraded.

Build instructions
------------
  - Clone the source:

        git clone git@github.com:apporfm-io/HBaseDS.git

  - Build

        mvn install


Dependency
------------

**Maven**
```
<dependency>
    <groupId>io.appform.hbase.ds</groupId>
    <artifactId>hbase-ds</artifactId>
    <version>0.0.1</version>
</dependency>
```
**Leiningen**
```
[io.appform.hbase.ds/hbase-ds "0.0.1"]
```

**Gradle**
```
compile "io.appform.hbase.ds:hbase-ds:0.0.1"
```
