### What is this?

This is a fork of the project [SimpleDiskCache] (https://github.com/fhucho/simple-disk-cache/) which implements the functionality which I need the most: an easy caching API for storing every object which implements java.io.Serializable.
SimpleDiskCache is a easy to use disk cache which uses [DiskLruCache](https://github.com/JakeWharton/DiskLruCache) under the hood. The DiskLruCache is nice, but it has too low level interface for most use cases. Besides the DiskLruCache, it also requires the [Apache Commons IO](http://commons.apache.org/proper/commons-io) lib (but it should be easy to remove this dependency).

### Quick intro

Every cache entry has a value and metadata, the metadata are representad as `Map<String, Serializable>`. Any string can be used as a cache key.

##### Open

Open the cache with `EveryObjectDiskCache.open(dir, appVersion, capacityBytes)`. This cannot be called twice with the same dir, otherwise `RuntimeException` will be thrown mercilessly.

##### Put

To put a serializable object to the cache, call `cache.put(key, serializableObject, metadata)`. Or just `cache.put(key, serializableObject)`. That's up to you.

##### Get

`cache.getObjectEntry(key)` returns `ObjectEntry` or `null`. `ObjectEntry` contains an object and the metadata.

##### Example


