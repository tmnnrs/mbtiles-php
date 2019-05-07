# mbtiles-php
Simple PHP script which can be used to load tiled map data (PBF) directly from a MBTiles file.

```php
<?php

  $z = $_GET['z'];
  $x = $_GET['x'];
  $y = $_GET['y'];

  try {
    $conn = new PDO("sqlite:output.mbtiles");

    $z = floatval($z);
    $y = floatval($y);
    $x = floatval($x);
    $flip = true;
    if ($flip) {
      $y = pow(2, $z) - 1 - $y;
    }

    $sql = "SELECT * FROM tiles WHERE zoom_level = $z AND tile_column = $x AND tile_row = $y";
    $q = $conn->prepare($sql);
    $q->execute();

    $q->bindColumn(1, $zoom_level);
    $q->bindColumn(2, $tile_column);
    $q->bindColumn(3, $tile_row);
    $q->bindColumn(4, $tile_data, PDO::PARAM_LOB);

    if ($data = $q->fetch()) {
      do {
        header('Access-Control-Allow-Origin: *');
        header('Content-Type: application/octet-stream');
        header('Content-Encoding: gzip');
        echo $tile_data;
      } while ($data = $q->fetch());
    }
    else {
      header('Access-Control-Allow-Origin: *');
      header('HTTP/1.1 204 No Content');
      header('Content-Type: application/json; charset=utf-8');
      echo '{ "message": "Tile does not exist" }';
    }
  }
  catch(PDOException $e) {
    print 'Exception : '.$e->getMessage();
  }

?>
```

The script can be used to add tile sources to a Mapbox style document as follows:

```javascript
"sources": {
  "localhost-streets": {
    "type": "vector",
    "tiles": [ "http://localhost:8080/vector-tiles/mbtiles.php?z={z}&x={x}&y={y}" ],
    "maxzoom": 14
  }
}
```
