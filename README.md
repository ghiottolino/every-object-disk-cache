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

<code>
@EBean(scope = Scope.Singleton)
public class CacheAwareTodoServiceImpl implements TodoService {

    private final int appVersion = 1;
    private File cacheDir;

    private EveryObjectDiskCache cache;

    @RootContext
    Context context;

    @Bean(BroadcastServiceImpl.class)
    TodoService todoService; //Inject it

    @AfterInject
    void initCache() {
        cacheDir = context.getCacheDir();
        try{
            //INITIALIZE FROM CACHE
            cache = EveryObjectDiskCache.open(cacheDir, appVersion, Integer.MAX_VALUE);
        } catch (IOException e) {
 	    // do exception handling
        }
    }

    @Override
    public List<Todo> getTodos(String username) {

        List<Todo> todos = new ArrayList<Todo>();
        try {
            String cacheKey = generateKey(username);

            //READ FROM CACHE
            EveryObjectDiskCache.ObjectEntry cachedObjectEntry = cache.getObjectEntry(cacheKey);


            if (cachedObjectEntry==null){
               todos = todoService.getTodos(username);
               //wrap List<Todo> in a TodoList objects, so that it is serializable
               TodoList todoList = new TodoList();
               todoList.setTodos(todos);
               //empty metadata object, to be used in the future for storing things like caching date
               Map<String,String> metadata = new HashMap<String, String>();
             
               //WRITE TO CACHE
               cache.put(cacheKey,todoList,metadata);

 	       //READ FROM CACHE
               cachedObjectEntry = cache.getObjectEntry(cacheKey);
            }
            TodoList todoList = (TodoList) cachedObjectEntry.getObject();
            todos = todoList.getTodos();
        } catch (IOException e) {
            e.printStackTrace();
        }

        return todos;
    }

    ...
}

</code>

