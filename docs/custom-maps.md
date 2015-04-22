# Custom Maps

## Extending the Map Class

You might want to extend the _Map_ class to provide convenience methods specific to your application. As with writing a custom matching rule, this is a bit involved but not difficult:

1. Write your _Map_ extended class.

2. Add your extended class factory to the _RouterContainer_.

3. Use the _RouterContainer_ as normal, and it will return your custom _Map_ class.

For example, you may wish to have a `resource()` method that automatically attaches a series of routes all at once for a given route name and route path.  First, write your extension of the _Map_ class:

```php
<?php
use Aura\Router\Map;

class MyResourceMap extends Map
{
    public function resource($namePrefix, $pathPrefix)
    {
        return $this->attach($namePrefix, $pathPrefix, function ($map) {
            $map->get('browse', '');
            $map->get('read', '/{id}');
            $map->patch('edit', '/{id}');
            $map->post('add', '');
            $map->delete('delete', '/{id}');
        });
    }
}
?>
```

Then tell the _RouterContainer_ how to create your extended class using a factory callable:

```php
<?php
use Aura\Router\Route;

$routerContainer->setMapFactory(function () {
    return new MyResourceMap(new Route());
});
?>
```

Now you can get the _Map_ from the _RouterContainer_, and it will be your custom extended class:

```php
<?php
$map = $routerContainer->getMap();
echo get_class($map); // "ResourceMap"
?>
```

## Caching Routes

You may wish to cache your route map for production deployments so that you do not have to add the routes from scratch on each page load. You can call the `RouterContainer::setMapBuilder()` method and pass a builder callable to set up the _Map_ before the container returns it. The builder callable should take a _Map_ instance as its only argument.

The following is a naive example for file-based caching and restoring of _Map_ routes. It uses the `Map::setRoutes()` and `Map::getRoutes()` methods to work with the array of mapped route objects.

```php
<?php
$routerContainer->setMapBuilder(function ($map) {

    // the cache file location
    $cache = '/path/to/routes.cache';

    // does the cache exist?
    if (file_exists($cache)) {

        // restore from the cache
        $routes = unserialize(file_get_contents($cache));
        $map->setRoutes($routes);

    } else {

        // build the routes on the map ...
        $map->get(...);
        $map->post(...);

        // ... then save them to the cache for the next page load
        $routes = $map->getRoutes();
        file_put_contents($cache, serialize($routes));
    }
});
?>
```

Now when we call `$routerContainer->getMap()`, the container will automatically call the map-builder logic and apply it to the `$map` before returning it.

Two notes:

1. If there are closures in the _Route_ objects (e.g. in the handlers), you will not be able to serialize the routes for caching. This is because closures cannot be serialized by PHP. Consider using non-closure callables instead.

2. The map-builder callable can be any callable: a closure, invokable object, etc.
