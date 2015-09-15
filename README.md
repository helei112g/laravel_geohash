# laravel_geohash
在laravel中使用geohash实现附近的功能

随着附近的X ，越来越实用。很多APP都加入了该功能，那么它该怎么实现？在php中如何使用。首先可以参考：

http://blog.dixo.net/downloads/geohash-php-class/

## 环境
laravel 5.8

## 使用距离
假设，使用手机获取到经纬度 30.555, 104.07，然后需要查找附近的软件公司。

```php
$lat = '30.555';
$long = '104.07';
$geohash = new GeoHash();

$hash = $geohash->encode($lat, $long);
// 决定查询范围，值越大，获取的范围越小
// 当geohash base32编码长度为8时，精度在19米左右，而当编码长度为9时，精度在2米左右，编码长度需要根据数据情况进行选择。
$pre_hash = substr($hash, 0, 5);

//取出相邻八个区域
$neighbors = $geohash->neighbors($pre_hash);
array_push($neighbors, $pre_hash);

$values = '';
foreach ($neighbors as $key=>$val) {
  $values .= '\'' . $val . '\'' .',';
}
$values = substr($values, 0, -1);
```

通过以上操作，会获取到：$values = "'wm6n0','wm6j8','wm6jc','wm3vz','wm3yp','wm6n1','wm6j9','wm3vx','wm6jb'";

拼接成这样的字符串主要是为了后面的数据库查询工作。

用sql查询操作：
```sql
SELECT * FROM `address` WHERE LEFT(`geohash`,5) IN ('wm6n0','wm6j8','wm6jc','wm3vz','wm3yp','wm6n1','wm6j9','wm3vx','wm6jb')
```

查找出来，范围是：
![这里写图片描述](http://img.blog.csdn.net/20150904141041421)

到这里，从数据库返回的结果，就是你附近的店，但是这时候并没有计算出实际距离与排序，还需要写代码计算实际距离并排序。


备注：这只是附近的点实现的一种技术方案。更推荐使用mongodb

其他资料：
* 原理解释：http://www.cnblogs.com/LBSer/p/3310455.html
* geohash演示: http://cevin.science/geohash/

