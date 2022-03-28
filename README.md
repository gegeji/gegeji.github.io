# 结果JSON定义（第2版）

参考 [jazzband/geojson: Python bindings and utilities for GeoJSON (github.com)](https://github.com/jazzband/geojson) 的结果，通过GeoJSON的properties结果传递参数。

> Ver 3修订说明
> 删除第二版中带❎的功能。新增街道、高德坐标系数据的返回。

> Ver. 2修订说明
> 由于 FeatureCollection 没有 properties 属性，每一个结果改成定义一个新类型表示。
> 
> 受到后端库的限制， FeatureCollection 必须以转换成json字符串的形式返回。前端在解析的时候需要重新构造 FeatureCollection 对象。

## 定义

缩进表示类型定义的层次结构。

带❎表示暂未实现这个功能（不知道这些属性有没有用？暂不实现）。

### QueryResponse

```diff
QueryResponse 最终返回的最外层结果对象
  statusMessage: String 返回状态数据，可以通过这个得知服务器返回的本次查询的状态数据
  geojsonResults: List[QuerySingleResult] 结果数组
```

### QuerySingleResult

```diff
QuerySingleResult 每一个计算结果（**由于FeatureCollection没有properties属性，属于每个结果的整体统计数据在这个对象上进行定义**）
  geojsonResult:String(FeatureCollection)  
    features:List[Feature[Point], Feature[LineString]] 每一个点、边
  averageEdgeDistance 平均每一条查询边的距离
  averageDistanceDeviation 平均距离误差
  averageAngleDeviation 平均角度误差
  district 这个解落在哪个区，返回第一个点所在的区
-  nearbyPoiCount: List[PoiCount]❎ 附近同类poi的数据（按照匹配分数查询）
  distanceToCurrentLocation 离定位中心点的距离 **（仅限于按距离计算的情况）**
  score 经过加权折算后的分数
  angleScore 原始的角度分数（100-angleDeviationPercent）
    最小是0（假如误差太大的话强制置为0），最大100，可以不以数字显示，以一个长条表示这个解的好坏
  distanceScore 原始的距离分数 （100-distanceDeviationPercent）
    最小是0（假如误差太大的话强制置为0），最大100，可以不以数字显示，以一个长条表示这个解的好坏 
  environmentScore 环境分数（指附近同类POI数量的情况）
    最小是0（假如误差太大的话强制置为0），最大100，可以不以数字显示，以一个长条表示这个解的好坏
```

### PoiCount❎

```diff
-  PoiCount❎ 表示某一个类poi的数量
-    poiType❎ poi类型
-   count❎ poi的数量
```

### Feature[LineString]

```diff
Feature[LineString] 表示查询边
  geometry: LineString
    coordinate: List[Position] 查询边的（两点）坐标，GeoJSON定义所必需的
  properties
    "stroke":"#ff0000" 绘制参数，用于GeoJSON.io上的绘制
    "stroke-width":3, 绘制参数，用于GeoJSON.io上的绘制
    "stroke-opacity":1, 绘制参数，用于GeoJSON.io上的绘制
    angleDeviationDegree 以角度表示的方向角误差（结果边与查询边两边之间以小于180度的角作为偏差角）
    angleDeviationPercent：以百分制表示的方向角误差（最大误差是180度，百分制的分数是180°，以与180°的比例作为百分制的方向角误差
    distanceDeviationMeter 以米表示的距离误差（通过绝对值表示成正值）
    distanceDeviationPercent 以百分制表示的距离误差（分母是查询边的长度）
    distance 最终结果边两点之间的距离
    poiIdx1: 第一个端点的下标
    poiIdx2: 第二个端点的下标
```

### Feature[Point]

```diff
Feature[Point] 表示每一个点
  geometry: Point
    Position 坐标（基于转换之后的WGS84，但是天地图的坐标系也接近于WGS84）
  properties
    poiMajorType POI的二级类型（对应于原来的title）
    poiMinorType POI的三级类型，它有时候比二级类型更加细致，如中餐馆下面区分了不同的餐馆
    address POI地址
    district POI所在的区
    name POI的名字
-   distanceInfo❎ 它连到的别的点的边的距离信息？相当于每条边的信息同时写在它的两个端点上面
    "marker-color":"#ff0000", 绘制参数，用于GeoJSON.io上的绘制
    "marker-size" :"medium", 绘制参数，用于GeoJSON.io上的绘制
+   gcjLatitude 高德坐标系下的纬度
+   gcjLongitude 高德坐标系下的经度
+   township 街道

```  

## 样例

以下这个例子返回的是整个QueryResponse本身。

```json
{
    "statusMessage": "Success.",
    "geojsonResults": [
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.163017,22.565963]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"莲塘国威中路\",\"name\":\"电信家属楼\",\"district\":\"罗湖区\",\"gcjLatitude\":22.5632,\"gcjLongitude\":114.168003,\"township\":\"莲塘街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.16451799999999,22.565877]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"特色/地方风味餐厅\",\"address\":\"莲塘村聚宝路77号(华润万家对面)\",\"name\":\"正新鸡排(宝驹店)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.563111,\"gcjLongitude\":114.1695,\"township\":\"莲塘街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.16603500000001,22.564870000000003]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场入口\",\"address\":\"莲塘村七巷\",\"name\":\"金色年华家园地下停车场(入口)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.562102,\"gcjLongitude\":114.17101299999999,\"township\":\"莲塘街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.163017,22.565963],[114.16451799999999,22.565877]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":5.361749699259406,\"angleDeviationPercent\":2.978749832921892,\"distanceDeviationMeter\":4.421809943248803,\"distanceDeviationPercent\":2.947873295499202,\"distance\":154.4218099432488,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.16451799999999,22.565877],[114.16603500000001,22.564870000000003]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":27.82938449651698,\"angleDeviationPercent\":15.460769164731655,\"distanceDeviationMeter\":6.1614734179593995,\"distanceDeviationPercent\":3.1118552615956565,\"distance\":191.8385265820406,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 173.1301682626447,
            "averageDistanceDeviation": 5.291641680604101,
            "averageAngleDeviation": 16.595567097888193,
            "district": "罗湖区",
            "score": 78.83596060256943,
            "angleScore": 90.78024050117322,
            "distanceScore": 96.97013572145258,
            "environmentScore": 18.67905056759546
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.09568200000001,22.579492000000002]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"玉龙新村A区55-57\",\"name\":\"梦君楼\",\"district\":\"罗湖区\",\"gcjLatitude\":22.576814000000002,\"gcjLongitude\":114.100806,\"township\":\"清水河街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.097016,22.578998000000002]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"四川菜(川菜)\",\"address\":\"玉龙新村A区86-1\",\"name\":\"飘香川菜馆(宝洁路)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.576319,\"gcjLongitude\":114.102139,\"township\":\"清水河街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.09827,22.577622]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"金湖路4号\",\"name\":\"金马花园停车场\",\"district\":\"罗湖区\",\"gcjLatitude\":22.574942999999998,\"gcjLongitude\":114.10339099999999,\"township\":\"清水河街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.09568200000001,22.579492000000002],[114.097016,22.578998000000002]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":22.40290432749245,\"angleDeviationPercent\":12.446057959718027,\"distanceDeviationMeter\":2.4311317026357813,\"distanceDeviationPercent\":1.6207544684238542,\"distance\":147.56886829736422,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.097016,22.578998000000002],[114.09827,22.577622]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":13.750072087492072,\"angleDeviationPercent\":7.638928937495596,\"distanceDeviationMeter\":1.9682176234797168,\"distanceDeviationPercent\":0.9940493047877358,\"distance\":199.96821762347972,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 173.76854296042197,
            "averageDistanceDeviation": 2.199674663057749,
            "averageAngleDeviation": 18.07648820749226,
            "district": "罗湖区",
            "score": 78.43217808882517,
            "angleScore": 89.95750655139318,
            "distanceScore": 98.69259811339421,
            "environmentScore": 14.860681114551083
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.108066,22.604371]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"格塘路99号\",\"name\":\"群芳楼(格塘路)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.601691,\"gcjLongitude\":114.113178,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.10946899999999,22.604371]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"布吉街道一村和兴街七号\",\"name\":\"营山谭家米粉店\",\"district\":\"龙岗区\",\"gcjLatitude\":22.601689999999998,\"gcjLongitude\":114.114579,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11115900000001,22.603610999999997]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"乐民路33号\",\"name\":\"布吉会堂地面停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.600929,\"gcjLongitude\":114.116266,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.108066,22.604371],[114.10946899999999,22.604371]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.082565279730858,\"angleDeviationPercent\":1.1569807109615877,\"distanceDeviationMeter\":5.977594859816264,\"distanceDeviationPercent\":3.985063239877509,\"distance\":144.02240514018374,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.10946899999999,22.604371],[114.11115900000001,22.603610999999997]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":37.192348234160306,\"angleDeviationPercent\":20.662415685644614,\"distanceDeviationMeter\":5.027296168913296,\"distanceDeviationPercent\":2.539038469148129,\"distance\":192.9727038310867,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 168.49755448563522,
            "averageDistanceDeviation": 5.50244551436478,
            "averageAngleDeviation": 19.637456756945582,
            "district": "龙岗区",
            "score": 77.50983495059707,
            "angleScore": 89.0903018016969,
            "distanceScore": 96.73794914548719,
            "environmentScore": 15.892672858617132
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.279195,22.722614]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"下井南六巷12号附近\",\"name\":\"吓井一村\",\"district\":\"龙岗区\",\"gcjLatitude\":22.719801,\"gcjLongitude\":114.28403200000001,\"township\":\"宝龙街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.280503,22.722191]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"广东菜(粤菜)\",\"address\":\"吓井南八巷15号\",\"name\":\"窑鸡王\",\"district\":\"龙岗区\",\"gcjLatitude\":22.719379999999997,\"gcjLongitude\":114.28534099999999,\"township\":\"宝龙街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.281663,22.720660000000002]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"龙湖南路56号附近\",\"name\":\"吓井一村停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.71785,\"gcjLongitude\":114.286501,\"township\":\"宝龙街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.279195,22.722614],[114.280503,22.722191]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":20.003487084095127,\"angleDeviationPercent\":11.113048380052849,\"distanceDeviationMeter\":7.838566056993528,\"distanceDeviationPercent\":5.225710704662352,\"distance\":142.16143394300647,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.280503,22.722191],[114.281663,22.720660000000002]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":8.55628434520527,\"angleDeviationPercent\":4.753491302891817,\"distanceDeviationMeter\":9.69416514299607,\"distanceDeviationPercent\":4.896043001513166,\"distance\":207.69416514299607,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 174.92779954300127,
            "averageDistanceDeviation": 8.7663655999948,
            "averageAngleDeviation": 14.279885714650199,
            "district": "龙岗区",
            "score": 77.09336299400259,
            "angleScore": 92.06673015852766,
            "distanceScore": 94.93912314691224,
            "environmentScore": 11.455108359133128
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.10970900000001,22.61286]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅区\",\"address\":\"长兴路与长龙路交叉口西150米\",\"name\":\"长龙3区\",\"district\":\"龙岗区\",\"gcjLatitude\":22.61018,\"gcjLongitude\":114.114819,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.111397,22.612845999999998]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"长龙1区六巷4号\",\"name\":\"饶洋粥饭店\",\"district\":\"龙岗区\",\"gcjLatitude\":22.610165,\"gcjLongitude\":114.116504,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11187199999999,22.611473]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"长龙路11号附近\",\"name\":\"乐乐停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.608792,\"gcjLongitude\":114.116978,\"township\":\"布吉街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.10970900000001,22.61286],[114.111397,22.612845999999998]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.557756347070381,\"angleDeviationPercent\":1.420975748372434,\"distanceDeviationMeter\":23.274881645907783,\"distanceDeviationPercent\":15.516587763938523,\"distance\":173.27488164590778,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.111397,22.612845999999998],[114.11187199999999,22.611473]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":9.51054635240422,\"angleDeviationPercent\":5.283636862446788,\"distanceDeviationMeter\":37.73248365721025,\"distanceDeviationPercent\":19.056809927883968,\"distance\":160.26751634278975,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 166.77119899434877,
            "averageDistanceDeviation": 30.503682651559018,
            "averageAngleDeviation": 6.0341513497373,
            "district": "龙岗区",
            "score": 76.63603880634473,
            "angleScore": 96.6476936945904,
            "distanceScore": 82.71330115408875,
            "environmentScore": 24.458204334365323
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.261641,22.722397]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"龙岗街道南联社区\",\"name\":\"吓岗一村\",\"district\":\"龙岗区\",\"gcjLatitude\":22.719568,\"gcjLongitude\":114.266471,\"township\":\"龙岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.262754,22.722396]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"湖南菜(湘菜)\",\"address\":\"碧新路龙岗段2065\",\"name\":\"湘益土灶柴火鱼\",\"district\":\"龙岗区\",\"gcjLatitude\":22.719568,\"gcjLongitude\":114.26758400000001,\"township\":\"龙岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.26371699999999,22.721133]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出入口\",\"address\":\"吓岗二村二巷\",\"name\":\"吓岗二小区停车场(出入口)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.718304999999997,\"gcjLongitude\":114.268547,\"township\":\"龙岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.261641,22.722397],[114.262754,22.722396]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.1340439537240172,\"angleDeviationPercent\":1.1855799742911206,\"distanceDeviationMeter\":35.8452133005282,\"distanceDeviationPercent\":23.8968088670188,\"distance\":114.1547866994718,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.262754,22.722396],[114.26371699999999,22.721133]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":8.730439909000111,\"angleDeviationPercent\":4.850244393888951,\"distanceDeviationMeter\":26.30602235263072,\"distanceDeviationPercent\":13.28586987506602,\"distance\":171.69397764736928,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 142.92438217342055,
            "averageDistanceDeviation": 31.07561782657946,
            "averageAngleDeviation": 5.432241931362064,
            "district": "龙岗区",
            "score": 76.37177925410802,
            "angleScore": 96.98208781590996,
            "distanceScore": 81.40866062895759,
            "environmentScore": 25.077399380804955
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.148793,22.620398]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅区\",\"address\":\"树山街3号\",\"name\":\"丹华居\",\"district\":\"龙岗区\",\"gcjLatitude\":22.617668,\"gcjLongitude\":114.153821,\"township\":\"南湾街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.150104,22.620351]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"丹竹头村丹竹路33号1楼\",\"name\":\"一鸣乳鸽·烧鹅王\",\"district\":\"龙岗区\",\"gcjLatitude\":22.617619,\"gcjLongitude\":114.155128,\"township\":\"南湾街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.151312,22.619177]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"沙平南路75号附近\",\"name\":\"停车场(丹河南路)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.616443,\"gcjLongitude\":114.156333,\"township\":\"南湾街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.148793,22.620398],[114.150104,22.620351]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":4.135768052365762,\"angleDeviationPercent\":2.297648917980979,\"distanceDeviationMeter\":15.335917419585314,\"distanceDeviationPercent\":10.22394494639021,\"distance\":134.66408258041469,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.150104,22.620351],[114.151312,22.619177]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":17.223763536218,\"angleDeviationPercent\":9.568757520121112,\"distanceDeviationMeter\":17.95753606162188,\"distanceDeviationPercent\":9.069462657384788,\"distance\":180.04246393837812,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 157.35327325939642,
            "averageDistanceDeviation": 16.646726740603597,
            "averageAngleDeviation": 10.679765794291882,
            "district": "龙岗区",
            "score": 76.26545721226441,
            "angleScore": 94.06679678094895,
            "distanceScore": 90.3532961981125,
            "environmentScore": 12.487100103199174
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.36673600000002,22.746913]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅区\",\"address\":\"坑梓镇龙田路口\",\"name\":\"梓龙楼\",\"district\":\"坪山区\",\"gcjLatitude\":22.744254,\"gcjLongitude\":114.371684,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.368572,22.746843]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"特色/地方风味餐厅\",\"address\":\"坪山新区坑梓街道梓兴路126号(大新百货对面多加福美食城内)\",\"name\":\"重庆冒菜(八分店)\",\"district\":\"坪山区\",\"gcjLatitude\":22.744186,\"gcjLongitude\":114.373522,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.36931299999999,22.745313]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出入口\",\"address\":\"梓兴路与育新街交叉口东南100米\",\"name\":\"陈氏香鹅掌停车场(出入口)\",\"district\":\"坪山区\",\"gcjLatitude\":22.742656,\"gcjLongitude\":114.37426299999998,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.36673600000002,22.746913],[114.368572,22.746843]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":4.265987345699585,\"angleDeviationPercent\":2.369992969833103,\"distanceDeviationMeter\":38.436302369267196,\"distanceDeviationPercent\":25.624201579511464,\"distance\":188.4363023692672,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.368572,22.746843],[114.36931299999999,22.745313]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.7524609391611534,\"angleDeviationPercent\":1.5291449662006409,\"distanceDeviationMeter\":11.672907917594074,\"distanceDeviationPercent\":5.8954080391889265,\"distance\":186.32709208240593,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 187.38169722583655,
            "averageDistanceDeviation": 25.054605143430635,
            "averageAngleDeviation": 3.5092241424303694,
            "district": "坪山区",
            "score": 76.23926390494586,
            "angleScore": 98.05043103198312,
            "distanceScore": 84.24019519064981,
            "environmentScore": 16.615067079463365
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.20681499999999,22.713647]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅区\",\"address\":\"大运新城清辉路与华美中路交汇处\",\"name\":\"美域蓝湾花园\",\"district\":\"龙岗区\",\"gcjLatitude\":22.710836999999998,\"gcjLongitude\":114.211701,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.20808500000001,22.713115]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"大围一村黄阁坑德为门诊部旁\",\"name\":\"明记小炒快餐宵夜麻辣烫\",\"district\":\"龙岗区\",\"gcjLatitude\":22.710303,\"gcjLongitude\":114.21296799999999,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.20922,22.711615]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"大运路与清辉路交叉口东南100米\",\"name\":\"停车场(大运路)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.708801,\"gcjLongitude\":114.214101,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.20681499999999,22.713647],[114.20808500000001,22.713115]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":24.81126006896403,\"angleDeviationPercent\":13.784033371646684,\"distanceDeviationMeter\":6.9313184896390965,\"distanceDeviationPercent\":4.620878993092731,\"distance\":143.0686815103609,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.20808500000001,22.713115],[114.20922,22.711615]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":8.519573567103578,\"angleDeviationPercent\":4.733096426168655,\"distanceDeviationMeter\":5.40436358024138,\"distanceDeviationPercent\":2.729476555677465,\"distance\":203.40436358024138,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 173.23652254530114,
            "averageDistanceDeviation": 6.167841034940238,
            "averageAngleDeviation": 16.665416818033805,
            "district": "龙岗区",
            "score": 76.06489302356215,
            "angleScore": 90.74143510109232,
            "distanceScore": 96.3248222256149,
            "environmentScore": 6.191950464396285
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.109527,22.544379]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"深南东路4003号\",\"name\":\"世金汉宫C座\",\"district\":\"罗湖区\",\"gcjLatitude\":22.541692,\"gcjLongitude\":114.11463300000001,\"township\":\"南湖街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11116200000001,22.544355]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"湖南菜(湘菜)\",\"address\":\"建设路2026-5号\",\"name\":\"湘里味道(建设路店)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.541667,\"gcjLongitude\":114.11626499999998,\"township\":\"南湖街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11289599999999,22.543963]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"人民南路3005号深房广场\",\"name\":\"深房广场停车场\",\"district\":\"罗湖区\",\"gcjLatitude\":22.541273,\"gcjLongitude\":114.11799599999999,\"township\":\"南湖街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.109527,22.544379],[114.11116200000001,22.544355]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.9235438466141375,\"angleDeviationPercent\":1.624191025896743,\"distanceDeviationMeter\":17.9322230529323,\"distanceDeviationPercent\":11.954815368621533,\"distance\":167.9322230529323,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.11116200000001,22.544355],[114.11289599999999,22.543963]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":48.66744180917874,\"angleDeviationPercent\":27.03746767176597,\"distanceDeviationMeter\":14.664602295473998,\"distanceDeviationPercent\":7.406364795693938,\"distance\":183.335397704526,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 175.63381037872915,
            "averageDistanceDeviation": 16.29841267420315,
            "averageAngleDeviation": 25.79549282789644,
            "district": "罗湖区",
            "score": 75.8443486362731,
            "angleScore": 85.66917065116864,
            "distanceScore": 90.31940991784226,
            "environmentScore": 27.24458204334365
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.22601,22.724432]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"白灰围二路66号\",\"name\":\"满园\",\"district\":\"龙岗区\",\"gcjLatitude\":22.721605,\"gcjLongitude\":114.23086299999999,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.22731499999999,22.72438]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"白灰围一路19号附近\",\"name\":\"侯彩播\",\"district\":\"龙岗区\",\"gcjLatitude\":22.721552,\"gcjLongitude\":114.23216699999999,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.228362,22.723112]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"吉福路111号附近\",\"name\":\"锦绣东方B区停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.720283,\"gcjLongitude\":114.23321200000001,\"township\":\"龙城街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.22601,22.724432],[114.22731499999999,22.72438]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":4.36440833352691,\"angleDeviationPercent\":2.4246712964038393,\"distanceDeviationMeter\":16.02990702991829,\"distanceDeviationPercent\":10.686604686612194,\"distance\":133.9700929700817,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.22731499999999,22.72438],[114.228362,22.723112]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":10.95281577787955,\"angleDeviationPercent\":6.084897654377528,\"distanceDeviationMeter\":20.76822465923385,\"distanceDeviationPercent\":10.48900235314841,\"distance\":177.23177534076615,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 155.60093415542394,
            "averageDistanceDeviation": 18.39906584457607,
            "averageAngleDeviation": 7.65861205570323,
            "district": "龙岗区",
            "score": 75.65223208775332,
            "angleScore": 95.7452155246093,
            "distanceScore": 89.41219648011969,
            "environmentScore": 7.946336429308566
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.111696,22.54685]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"永新街57-15号永新商业城\",\"name\":\"永新商业城\",\"district\":\"罗湖区\",\"gcjLatitude\":22.544162,\"gcjLongitude\":114.116798,\"township\":\"东门街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11355,22.546826]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中式素菜馆\",\"address\":\"东门新干线4层\",\"name\":\"云来居素食馆(东门店)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.544135999999998,\"gcjLongitude\":114.11865,\"township\":\"东门街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.11392,22.545484]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"深南东路3039号创意嘉年华\",\"name\":\"创意嘉年华停车场\",\"district\":\"罗湖区\",\"gcjLatitude\":22.542794,\"gcjLongitude\":114.119019,\"township\":\"南湖街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.111696,22.54685],[114.11355,22.546826]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.8242167934273255,\"angleDeviationPercent\":1.5690093296818475,\"distanceDeviationMeter\":40.41716111812275,\"distanceDeviationPercent\":26.9447740787485,\"distance\":190.41716111812275,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.11355,22.546826],[114.11392,22.545484]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":13.180052456823375,\"angleDeviationPercent\":7.3222513649018754,\"distanceDeviationMeter\":44.01438946105182,\"distanceDeviationPercent\":22.22948962679385,\"distance\":153.98561053894818,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 172.20138582853548,
            "averageDistanceDeviation": 42.215775289587285,
            "averageAngleDeviation": 8.00213462512535,
            "district": "罗湖区",
            "score": 75.63147716331844,
            "angleScore": 95.55436965270815,
            "distanceScore": 75.41286814722882,
            "environmentScore": 36.22291021671827
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.10198000000001,22.621791]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"社区中心\",\"address\":\"新梅子园八巷与梅园六巷交叉口东南200米\",\"name\":\"爱尚家社区服务站\",\"district\":\"龙岗区\",\"gcjLatitude\":22.619117000000003,\"gcjLongitude\":114.1071,\"township\":\"吉华街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.103353,22.621768]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"布吉街道上水径老围90号1楼\",\"name\":\"梅州小食馆\",\"district\":\"龙岗区\",\"gcjLatitude\":22.619093,\"gcjLongitude\":114.10847199999999,\"township\":\"吉华街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.10458899999999,22.620666]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"上水径市场二街与官坑一街交叉口东100米\",\"name\":\"上水径村停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.617991,\"gcjLongitude\":114.109706,\"township\":\"吉华街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.10198000000001,22.621791],[114.103353,22.621768]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":3.0422737164045373,\"angleDeviationPercent\":1.6901520646691874,\"distanceDeviationMeter\":9.05181962384799,\"distanceDeviationPercent\":6.03454641589866,\"distance\":140.948180376152,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.103353,22.621768],[114.10458899999999,22.620666]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":19.68625363342349,\"angleDeviationPercent\":10.936807574124161,\"distanceDeviationMeter\":21.620496629277653,\"distanceDeviationPercent\":10.91944274205942,\"distance\":176.37950337072235,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 158.66384187343718,
            "averageDistanceDeviation": 15.336158126562822,
            "averageAngleDeviation": 11.364263674914014,
            "district": "龙岗区",
            "score": 75.50795884746086,
            "angleScore": 93.68652018060332,
            "distanceScore": 91.52300542102097,
            "environmentScore": 7.120743034055728
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.377362,22.750332999999998]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"坪山新区坑梓街道人民西路137号\",\"name\":\"幸福时光(宝梓北路)\",\"district\":\"坪山区\",\"gcjLatitude\":22.747685,\"gcjLongitude\":114.38231699999999,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.378736,22.750234]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"特色/地方风味餐厅\",\"address\":\"坪山坑梓镇人民路145号(近星晨宾馆)\",\"name\":\"大雁饺子馆(人民路店)\",\"district\":\"坪山区\",\"gcjLatitude\":22.747587,\"gcjLongitude\":114.383692,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.37921499999999,22.748819]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出入口\",\"address\":\"宝红路与红岭路交叉口西150米\",\"name\":\"新金凤酒楼停车场(出入口)\",\"district\":\"坪山区\",\"gcjLatitude\":22.746172,\"gcjLongitude\":114.384171,\"township\":\"坑梓街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.377362,22.750332999999998],[114.378736,22.750234]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":6.203741887218399,\"angleDeviationPercent\":3.4465232706768885,\"distanceDeviationMeter\":8.675139300718627,\"distanceDeviationPercent\":5.783426200479084,\"distance\":141.32486069928137,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.378736,22.750234],[114.37921499999999,22.748819]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":9.892227453491486,\"angleDeviationPercent\":5.4956819186063814,\"distanceDeviationMeter\":33.17021298958937,\"distanceDeviationPercent\":16.752632823024936,\"distance\":164.82978701041063,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 153.077323854846,
            "averageDistanceDeviation": 20.922676145154,
            "averageAngleDeviation": 8.047984670354943,
            "district": "坪山区",
            "score": 75.37617378282954,
            "angleScore": 95.52889740535836,
            "distanceScore": 88.73197048824798,
            "environmentScore": 8.359133126934983
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.174403,22.645738]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"横岗街道红棉路\",\"name\":\"红棉社区\",\"district\":\"龙岗区\",\"gcjLatitude\":22.642965,\"gcjLongitude\":114.17936399999999,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.175909,22.645474]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"清真菜馆\",\"address\":\"梧桐路5号附近\",\"name\":\"中国兰州拉面\",\"district\":\"龙岗区\",\"gcjLatitude\":22.642699,\"gcjLongitude\":114.180866,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.17642099999999,22.644002]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"振业大道\",\"name\":\"振业城二至五期停车场\",\"district\":\"龙岗区\",\"gcjLatitude\":22.641225,\"gcjLongitude\":114.181376,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.174403,22.645738],[114.175909,22.645474]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":12.02542146804393,\"angleDeviationPercent\":6.680789704468851,\"distanceDeviationMeter\":7.312546921186538,\"distanceDeviationPercent\":4.875031280791025,\"distance\":157.31254692118654,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.175909,22.645474],[114.17642099999999,22.644002]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":9.414996793430092,\"angleDeviationPercent\":5.230553774127829,\"distanceDeviationMeter\":26.09410023986598,\"distanceDeviationPercent\":13.178838504982817,\"distance\":171.90589976013402,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 164.60922334066026,
            "averageDistanceDeviation": 16.70332358052626,
            "averageAngleDeviation": 10.720209130737011,
            "district": "龙岗区",
            "score": 74.56423288892157,
            "angleScore": 94.04432826070166,
            "distanceScore": 90.97306510711309,
            "environmentScore": 2.786377708978328
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.089413,22.539193]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"松岭路7号\",\"name\":\"燃气公司小区\",\"district\":\"福田区\",\"gcjLatitude\":22.536510999999997,\"gcjLongitude\":114.09453899999998,\"township\":\"南园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.090349,22.539178]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"海鲜酒楼\",\"address\":\"东园路89-3号(同仁四季椰子鸡对面)\",\"name\":\"海鲜烧烤(东园路店)\",\"district\":\"福田区\",\"gcjLatitude\":22.536496,\"gcjLongitude\":114.095474,\"township\":\"南园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.09088600000001,22.538938]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出口\",\"address\":\"上步南路1001\",\"name\":\"锦峰大厦停车场(出口)\",\"district\":\"福田区\",\"gcjLatitude\":22.536255999999998,\"gcjLongitude\":114.096011,\"township\":\"南园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.089413,22.539193],[114.090349,22.539178]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":3.0006882821299996,\"angleDeviationPercent\":1.6670490456277776,\"distanceDeviationMeter\":53.85669901784877,\"distanceDeviationPercent\":35.904466011899174,\"distance\":96.14330098215123,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.090349,22.539178],[114.09088600000001,22.538938]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":37.32482014282141,\"angleDeviationPercent\":20.73601119045634,\"distanceDeviationMeter\":136.73170020271877,\"distanceDeviationPercent\":69.05641424379736,\"distance\":61.26829979728122,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 78.70580038971623,
            "averageDistanceDeviation": 95.29419961028377,
            "averageAngleDeviation": 20.162754212475704,
            "district": "福田区",
            "score": 74.52721190164387,
            "angleScore": 88.79846988195794,
            "distanceScore": 47.51955987215173,
            "environmentScore": 100
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.19854,22.648723]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"横岗镇裕民街\",\"name\":\"华西新光小区\",\"district\":\"龙岗区\",\"gcjLatitude\":22.645909,\"gcjLongitude\":114.20343999999999,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.200465,22.648713]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"特色/地方风味餐厅\",\"address\":\"龙岗大道段4207号(志健广场公交站前行约20米)\",\"name\":\"横岗诸葛烤鱼吧\",\"district\":\"龙岗区\",\"gcjLatitude\":22.645896,\"gcjLongitude\":114.20536100000001,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.201242,22.647261999999998]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出入口\",\"address\":\"坝心街4巷附近\",\"name\":\"坝心村停车场(出入口)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.644444,\"gcjLongitude\":114.20613600000001,\"township\":\"横岗街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.19854,22.648723],[114.200465,22.648713]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":2.3802030154430724,\"angleDeviationPercent\":1.3223350085794847,\"distanceDeviationMeter\":47.546746599725594,\"distanceDeviationPercent\":31.69783106648373,\"distance\":197.5467465997256,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.200465,22.648713],[114.201242,22.647261999999998]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":0.4252776090660859,\"angleDeviationPercent\":0.23626533837004773,\"distanceDeviationMeter\":18.028407580236063,\"distanceDeviationPercent\":9.105256353654577,\"distance\":179.97159241976394,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 188.75916950974477,
            "averageDistanceDeviation": 32.78757708998083,
            "averageAngleDeviation": 1.4027403122545792,
            "district": "龙岗区",
            "score": 74.45851899973002,
            "angleScore": 99.22069982652523,
            "distanceScore": 79.59845628993085,
            "environmentScore": 14.654282765737875
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.120347,22.561637]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"田贝一路64号附近\",\"name\":\"龙屋小区2栋\",\"district\":\"罗湖区\",\"gcjLatitude\":22.558942000000002,\"gcjLongitude\":114.12543600000001,\"township\":\"翠竹街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.12163799999999,22.561484]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"特色/地方风味餐厅\",\"address\":\"田贝一路19号\",\"name\":\"趣吃烧烤店\",\"district\":\"罗湖区\",\"gcjLatitude\":22.558787,\"gcjLongitude\":114.126724,\"township\":\"翠竹街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.123346,22.560144]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"公共停车场\",\"address\":\"东门北路1017号深圳市人民医院内\",\"name\":\"深圳市人民医院停车场(雅园立交)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.557445,\"gcjLongitude\":114.128429,\"township\":\"翠竹街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.120347,22.561637],[114.12163799999999,22.561484]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":8.84132221276272,\"angleDeviationPercent\":4.911845673757067,\"distanceDeviationMeter\":16.34635587681487,\"distanceDeviationPercent\":10.897570584543246,\"distance\":133.65364412318513,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.12163799999999,22.561484],[114.123346,22.560144]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":23.290274377559058,\"angleDeviationPercent\":12.939041320866144,\"distanceDeviationMeter\":32.13483789654737,\"distanceDeviationPercent\":16.229716109367356,\"distance\":230.13483789654737,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 181.89424100986625,
            "averageDistanceDeviation": 24.240596886681118,
            "averageAngleDeviation": 16.06579829516089,
            "district": "罗湖区",
            "score": 74.4099380177112,
            "angleScore": 91.0745565026884,
            "distanceScore": 86.4363566530447,
            "environmentScore": 17.027863777089784
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.223219,22.67922]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"横岗街道长江埔路76号\",\"name\":\"森雅谷\",\"district\":\"龙岗区\",\"gcjLatitude\":22.676384,\"gcjLongitude\":114.22807399999999,\"township\":\"园山街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.224652,22.679123]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"中餐厅\",\"address\":\"横岗街道横岗荷坳长江浦路70-2\",\"name\":\"华林达电商园餐厅\",\"district\":\"龙岗区\",\"gcjLatitude\":22.676285999999998,\"gcjLongitude\":114.229504,\"township\":\"园山街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.22520800000001,22.676935]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出入口\",\"address\":\"长江埔路49号\",\"name\":\"京铁科技工业园停车场(出入口)\",\"district\":\"龙岗区\",\"gcjLatitude\":22.674097,\"gcjLongitude\":114.23006000000001,\"township\":\"园山街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.223219,22.67922],[114.224652,22.679123]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":5.955018527922476,\"angleDeviationPercent\":3.3083436266235977,\"distanceDeviationMeter\":2.5829733114605062,\"distanceDeviationPercent\":1.7219822076403375,\"distance\":147.4170266885395,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.224652,22.679123],[114.22520800000001,22.676935]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":14.336162964674429,\"angleDeviationPercent\":7.964534980374682,\"distanceDeviationMeter\":51.89289559433769,\"distanceDeviationPercent\":26.208533128453375,\"distance\":249.8928955943377,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 198.6549611414386,
            "averageDistanceDeviation": 27.237934452899097,
            "averageAngleDeviation": 10.145590746298453,
            "district": "龙岗区",
            "score": 74.28522420415766,
            "angleScore": 94.36356069650085,
            "distanceScore": 86.03474233195314,
            "environmentScore": 10.62951496388029
        },
        {
            "geojsonResult": "{\"type\":\"FeatureCollection\",\"features\":[{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.103458,22.541438]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"住宅区\",\"poiMinorType\":\"住宅小区\",\"address\":\"书城路\",\"name\":\"蔡屋围新八·九坊\",\"district\":\"罗湖区\",\"gcjLatitude\":22.538754,\"gcjLongitude\":114.108571,\"township\":\"桂园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.104203,22.541420000000002]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"中餐厅\",\"poiMinorType\":\"湖南菜(湘菜)\",\"address\":\"蔡屋围新七坊八栋102\",\"name\":\"天园香湘味菜馆\",\"district\":\"罗湖区\",\"gcjLatitude\":22.538736,\"gcjLongitude\":114.109315,\"township\":\"桂园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"Point\",\"coordinates\":[114.104487,22.539687]},\"properties\":{\"marker-color\":\"#ff0000\",\"marker-size\":\"medium\",\"poiMajorType\":\"停车场\",\"poiMinorType\":\"停车场出口\",\"address\":\"宝安南路1001号华瑞大厦\",\"name\":\"华瑞大厦停车场(出口)\",\"district\":\"罗湖区\",\"gcjLatitude\":22.537003,\"gcjLongitude\":114.109599,\"township\":\"桂园街道\"}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.103458,22.541438],[114.104203,22.541420000000002]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":3.4666235620121313,\"angleDeviationPercent\":1.9259019788956284,\"distanceDeviationMeter\":73.46230020403584,\"distanceDeviationPercent\":48.974866802690556,\"distance\":76.53769979596416,\"poiIdx1\":0,\"poiIdx2\":1}},{\"type\":\"Feature\",\"geometry\":{\"type\":\"LineString\",\"coordinates\":[[114.104203,22.541420000000002],[114.104487,22.539687]]},\"properties\":{\"stroke\":\"#ff0000\",\"stroke-width\":3,\"stroke-opacity\":1,\"angleDeviationDegree\":19.287231526785746,\"angleDeviationPercent\":10.715128625992081,\"distanceDeviationMeter\":3.104079081506626,\"distanceDeviationPercent\":1.5677167078316294,\"distance\":194.89592091849337,\"poiIdx1\":1,\"poiIdx2\":2}}]}",
            "averageEdgeDistance": 135.71681035722878,
            "averageDistanceDeviation": 38.283189642771234,
            "averageAngleDeviation": 11.376927544398939,
            "district": "罗湖区",
            "score": 74.17442268775395,
            "angleScore": 93.67948469755615,
            "distanceScore": 74.7287082447389,
            "environmentScore": 34.05572755417957
        }
    ]
}
```

以下json表示的是GeoJSON结果的列表。

```json
[
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.09318700000001,
                        22.541317000000003
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "南园路埔尾村71号2楼新城市广对面亚洲眼镜2楼",
                    "name": "Jay Hair",
                    "district": "福田区",
                    "gcjLatitude": 22.538636,
                    "gcjLongitude": 114.09831000000001,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.09175,
                        22.5413
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "上步南路永富楼",
                    "name": "唯客铁板烧自选快餐(上步店)",
                    "district": "福田区",
                    "gcjLatitude": 22.538618,
                    "gcjLongitude": 114.096874,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.092421,
                        22.539848000000003
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "滨江新村22号801",
                    "name": "深圳市百树文化传播有限公司",
                    "district": "福田区",
                    "gcjLatitude": 22.537166,
                    "gcjLongitude": 114.097545,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.09318700000001,
                            22.541317000000003
                        ],
                        [
                            114.09175,
                            22.5413
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 30.881797055253006,
                    "angleDeviationPercent": 17.156553919585004,
                    "distanceDeviationMeter": 2.407792988068991,
                    "distanceDeviationPercent": 1.6051953253793272,
                    "distance": 147.592207011931,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.09175,
                            22.5413
                        ],
                        [
                            114.092421,
                            22.539848000000003
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 41.14246323993177,
                    "angleDeviationPercent": 22.856924022184316,
                    "distanceDeviationMeter": 2.5468060809250517,
                    "distanceDeviationPercent": 1.4721422433092783,
                    "distance": 175.54680608092505,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07856000000001,
                        22.535808
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福星路赤尾坊4栋1楼",
                    "name": "文文发艺",
                    "district": "福田区",
                    "gcjLatitude": 22.533122,
                    "gcjLongitude": 114.083688,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.077124,
                        22.535746
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "火锅店",
                    "address": "华强南福星路108号",
                    "name": "荔园四季椰子鸡(福星店)",
                    "district": "福田区",
                    "gcjLatitude": 22.533058999999998,
                    "gcjLongitude": 114.082252,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07864199999999,
                        22.535256
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福祥街赤尾坊12-4",
                    "name": "深港众一昊学车",
                    "district": "福田区",
                    "gcjLatitude": 22.53257,
                    "gcjLongitude": 114.08377,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.07856000000001,
                            22.535808
                        ],
                        [
                            114.077124,
                            22.535746
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 29.08734828317121,
                    "angleDeviationPercent": 16.159637935095116,
                    "distanceDeviationMeter": 2.3556361976211804,
                    "distanceDeviationPercent": 1.5704241317474537,
                    "distance": 147.64436380237882,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.077124,
                            22.535746
                        ],
                        [
                            114.07864199999999,
                            22.535256
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 6.165152685091925,
                    "angleDeviationPercent": 3.4250848250510697,
                    "distanceDeviationMeter": 7.848050384276121,
                    "distanceDeviationPercent": 4.536445308830127,
                    "distance": 165.15194961572388,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07856000000001,
                        22.535808
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福星路赤尾坊4栋1楼",
                    "name": "文文发艺",
                    "district": "福田区",
                    "gcjLatitude": 22.533122,
                    "gcjLongitude": 114.083688,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.077124,
                        22.535746
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "火锅店",
                    "address": "华强南福星路108号",
                    "name": "荔园四季椰子鸡(福星店)",
                    "district": "福田区",
                    "gcjLatitude": 22.533058999999998,
                    "gcjLongitude": 114.082252,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07864199999999,
                        22.535256
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福祥街赤尾坊12-4",
                    "name": "深港众一昊学车",
                    "district": "福田区",
                    "gcjLatitude": 22.53257,
                    "gcjLongitude": 114.08377,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.07856000000001,
                            22.535808
                        ],
                        [
                            114.077124,
                            22.535746
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 29.08734828317121,
                    "angleDeviationPercent": 16.159637935095116,
                    "distanceDeviationMeter": 2.3556361976211804,
                    "distanceDeviationPercent": 1.5704241317474537,
                    "distance": 147.64436380237882,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.077124,
                            22.535746
                        ],
                        [
                            114.07864199999999,
                            22.535256
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 6.165152685091925,
                    "angleDeviationPercent": 3.4250848250510697,
                    "distanceDeviationMeter": 7.848050384276121,
                    "distanceDeviationPercent": 4.536445308830127,
                    "distance": 165.15194961572388,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.12796200000001,
                        22.549818
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "凤凰路181-l号附近",
                    "name": "剪家班美发造型烫染",
                    "district": "罗湖区",
                    "gcjLatitude": 22.547113,
                    "gcjLongitude": 114.133035,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.126576,
                        22.549345000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "工纺大厦",
                    "name": "朱记小吃馆",
                    "district": "罗湖区",
                    "gcjLatitude": 22.546641,
                    "gcjLongitude": 114.131652,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.128196,
                        22.549318
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "东清平路11号",
                    "name": "长沙勘察院深圳分院",
                    "district": "罗湖区",
                    "gcjLatitude": 22.546612,
                    "gcjLongitude": 114.13326799999999,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.12796200000001,
                            22.549818
                        ],
                        [
                            114.126576,
                            22.549345000000002
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 12.716360694672009,
                    "angleDeviationPercent": 7.0646448303733385,
                    "distanceDeviationMeter": 1.7405437554621983,
                    "distanceDeviationPercent": 1.1603625036414655,
                    "distance": 151.7405437554622,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.126576,
                            22.549345000000002
                        ],
                        [
                            114.128196,
                            22.549318
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 23.10003535272969,
                    "angleDeviationPercent": 12.833352973738718,
                    "distanceDeviationMeter": 6.608350212495509,
                    "distanceDeviationPercent": 3.8198556141592537,
                    "distance": 166.3916497875045,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078246,
                        22.541388
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路世界贸易广场B座14C(华强路地铁站C出口)",
                    "name": "巴厘岛国际SPA",
                    "district": "福田区",
                    "gcjLatitude": 22.538702,
                    "gcjLongitude": 114.083375,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.078246,
                            22.541388
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 4.546272221462065,
                    "angleDeviationPercent": 2.525706789701147,
                    "distanceDeviationMeter": 11.297744690032658,
                    "distanceDeviationPercent": 7.531829793355106,
                    "distance": 161.29774469003266,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07860900000001,
                        22.535763
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "滨河大道出口与滨河大道辅道交叉口西北50米",
                    "name": "美琪丝(滨河大道辅道)",
                    "district": "福田区",
                    "gcjLatitude": 22.533077,
                    "gcjLongitude": 114.08373700000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.077124,
                        22.535746
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "火锅店",
                    "address": "华强南福星路108号",
                    "name": "荔园四季椰子鸡(福星店)",
                    "district": "福田区",
                    "gcjLatitude": 22.533058999999998,
                    "gcjLongitude": 114.082252,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07864199999999,
                        22.535256
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福祥街赤尾坊12-4",
                    "name": "深港众一昊学车",
                    "district": "福田区",
                    "gcjLatitude": 22.53257,
                    "gcjLongitude": 114.08377,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.07860900000001,
                            22.535763
                        ],
                        [
                            114.077124,
                            22.535746
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 30.903703441390455,
                    "angleDeviationPercent": 17.16872413410581,
                    "distanceDeviationMeter": 2.5275664470873664,
                    "distanceDeviationPercent": 1.6850442980582443,
                    "distance": 152.52756644708737,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.077124,
                            22.535746
                        ],
                        [
                            114.07864199999999,
                            22.535256
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 6.165152685091925,
                    "angleDeviationPercent": 3.4250848250510697,
                    "distanceDeviationMeter": 7.848050384276121,
                    "distanceDeviationPercent": 4.536445308830127,
                    "distance": 165.15194961572388,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07860900000001,
                        22.535763
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "滨河大道出口与滨河大道辅道交叉口西北50米",
                    "name": "美琪丝(滨河大道辅道)",
                    "district": "福田区",
                    "gcjLatitude": 22.533077,
                    "gcjLongitude": 114.08373700000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.077124,
                        22.535746
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "火锅店",
                    "address": "华强南福星路108号",
                    "name": "荔园四季椰子鸡(福星店)",
                    "district": "福田区",
                    "gcjLatitude": 22.533058999999998,
                    "gcjLongitude": 114.082252,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07864199999999,
                        22.535256
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福祥街赤尾坊12-4",
                    "name": "深港众一昊学车",
                    "district": "福田区",
                    "gcjLatitude": 22.53257,
                    "gcjLongitude": 114.08377,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.07860900000001,
                            22.535763
                        ],
                        [
                            114.077124,
                            22.535746
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 30.903703441390455,
                    "angleDeviationPercent": 17.16872413410581,
                    "distanceDeviationMeter": 2.5275664470873664,
                    "distanceDeviationPercent": 1.6850442980582443,
                    "distance": 152.52756644708737,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.077124,
                            22.535746
                        ],
                        [
                            114.07864199999999,
                            22.535256
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 6.165152685091925,
                    "angleDeviationPercent": 3.4250848250510697,
                    "distanceDeviationMeter": 7.848050384276121,
                    "distanceDeviationPercent": 4.536445308830127,
                    "distance": 165.15194961572388,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.07824699999999,
                        22.541395
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路9世界贸易广场B座荟景豪庭B-12B1",
                    "name": "美约会·美颜纤体中心",
                    "district": "福田区",
                    "gcjLatitude": 22.538709,
                    "gcjLongitude": 114.083376,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.07824699999999,
                            22.541395
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 4.332185446670604,
                    "angleDeviationPercent": 2.40676969259478,
                    "distanceDeviationMeter": 11.7650126669933,
                    "distanceDeviationPercent": 7.843341777995533,
                    "distance": 161.7650126669933,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078154,
                        22.541577
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路世界贸易广场A1706号",
                    "name": "多米潮牌造型全国连锁(华强北店)",
                    "district": "福田区",
                    "gcjLatitude": 22.538891,
                    "gcjLongitude": 114.08328300000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.078154,
                            22.541577
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 3.188943560533893,
                    "angleDeviationPercent": 1.7716353114077186,
                    "distanceDeviationMeter": 14.7897750122184,
                    "distanceDeviationPercent": 9.8598500081456,
                    "distance": 164.7897750122184,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.09294299999999,
                        22.541316000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "南园路埔尾村74号二楼主楼202(近新城市广场)",
                    "name": "卡顿(中信店)",
                    "district": "福田区",
                    "gcjLatitude": 22.538635,
                    "gcjLongitude": 114.09806599999999,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.09175,
                        22.5413
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "上步南路永富楼",
                    "name": "唯客铁板烧自选快餐(上步店)",
                    "district": "福田区",
                    "gcjLatitude": 22.538618,
                    "gcjLongitude": 114.096874,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.092421,
                        22.539848000000003
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "滨江新村22号801",
                    "name": "深圳市百树文化传播有限公司",
                    "district": "福田区",
                    "gcjLatitude": 22.537166,
                    "gcjLongitude": 114.097545,
                    "township": "南园街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.09294299999999,
                            22.541316000000002
                        ],
                        [
                            114.09175,
                            22.5413
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 30.79120592547192,
                    "angleDeviationPercent": 17.106225514151067,
                    "distanceDeviationMeter": 27.46579512006504,
                    "distanceDeviationPercent": 18.310530080043357,
                    "distance": 122.53420487993496,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.09175,
                            22.5413
                        ],
                        [
                            114.092421,
                            22.539848000000003
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 41.14246323993177,
                    "angleDeviationPercent": 22.856924022184316,
                    "distanceDeviationMeter": 2.5468060809250517,
                    "distanceDeviationPercent": 1.4721422433092783,
                    "distance": 175.54680608092505,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.116257,
                        22.542126
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "南湖路2050-2号",
                    "name": "人气专业造型",
                    "district": "罗湖区",
                    "gcjLatitude": 22.539433,
                    "gcjLongitude": 114.121352,
                    "township": "南湖街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.114751,
                        22.542118
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "南湖街道金光华广场四楼",
                    "name": "台湾一茶一坐茶餐厅",
                    "district": "罗湖区",
                    "gcjLatitude": 22.539427,
                    "gcjLongitude": 114.119848,
                    "township": "南湖街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.116257,
                        22.542092999999998
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "金光华广场斜对面",
                    "name": "深圳市盛世唐朝文化传播有限公司",
                    "district": "罗湖区",
                    "gcjLatitude": 22.5394,
                    "gcjLongitude": 114.121352,
                    "township": "南湖街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.116257,
                            22.542126
                        ],
                        [
                            114.114751,
                            22.542118
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 31.255228883544603,
                    "angleDeviationPercent": 17.36401604641367,
                    "distanceDeviationMeter": 4.668090726585945,
                    "distanceDeviationPercent": 3.11206048439063,
                    "distance": 154.66809072658594,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.114751,
                            22.542118
                        ],
                        [
                            114.116257,
                            22.542092999999998
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 23.103838801124454,
                    "angleDeviationPercent": 12.835466000624695,
                    "distanceDeviationMeter": 18.30948703460632,
                    "distanceDeviationPercent": 10.58351851711348,
                    "distance": 154.69051296539368,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.235004,
                        22.722095
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "清林东路85号紫薇花园西一栋(博士眼镜旁,紫薇花园左手边50米)",
                    "name": "闺蜜美容美甲",
                    "district": "龙岗区",
                    "gcjLatitude": 22.719263,
                    "gcjLongitude": 114.239846,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.233977,
                        22.721466
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "吉祥路390号",
                    "name": "膳心餐厅(吉祥路)",
                    "district": "龙岗区",
                    "gcjLatitude": 22.718633999999998,
                    "gcjLongitude": 114.23881999999999,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.23555800000001,
                        22.720848999999998
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "龙城街道紫微花园东7栋104室",
                    "name": "深圳足迹文化传播有限公司",
                    "district": "龙岗区",
                    "gcjLatitude": 22.718016,
                    "gcjLongitude": 114.2404,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.235004,
                            22.722095
                        ],
                        [
                            114.233977,
                            22.721466
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.07363809000679566,
                    "angleDeviationPercent": 0.040910050003775365,
                    "distanceDeviationMeter": 23.55939150782106,
                    "distanceDeviationPercent": 15.706261005214042,
                    "distance": 126.44060849217894,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.233977,
                            22.721466
                        ],
                        [
                            114.23555800000001,
                            22.720848999999998
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 2.736197247720412,
                    "angleDeviationPercent": 1.5201095820668955,
                    "distanceDeviationMeter": 3.0730983386621915,
                    "distanceDeviationPercent": 1.7763574211920183,
                    "distance": 176.0730983386622,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078239,
                        22.541021
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路5号中电福华大厦",
                    "name": "舒敏养生减肥",
                    "district": "福田区",
                    "gcjLatitude": 22.538335,
                    "gcjLongitude": 114.083368,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.078239,
                            22.541021
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 17.839127374178275,
                    "angleDeviationPercent": 9.91062631898793,
                    "distanceDeviationMeter": 4.679797273125047,
                    "distanceDeviationPercent": 3.1198648487500313,
                    "distance": 145.32020272687495,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078239,
                        22.541019
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路5号福华大厦201号",
                    "name": "深圳市东基公司金娜美容美发中心",
                    "district": "福田区",
                    "gcjLatitude": 22.538332999999998,
                    "gcjLongitude": 114.083368,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.078239,
                            22.541019
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 17.918207334026782,
                    "angleDeviationPercent": 9.954559630014879,
                    "distanceDeviationMeter": 4.736462460183759,
                    "distanceDeviationPercent": 3.1576416401225065,
                    "distance": 145.26353753981624,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078239,
                        22.541019
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "福虹路5中电福华大厦2层",
                    "name": "爱诗伦思美容形体设计中心",
                    "district": "福田区",
                    "gcjLatitude": 22.538332999999998,
                    "gcjLongitude": 114.083368,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.076871,
                        22.540687
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "福华路110号广业大厦2楼",
                    "name": "青藏羔羊",
                    "district": "福田区",
                    "gcjLatitude": 22.538,
                    "gcjLongitude": 114.08200000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.078581,
                        22.539935999999997
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "福华路福侨大厦A栋6楼",
                    "name": "深圳美尔施",
                    "district": "福田区",
                    "gcjLatitude": 22.53725,
                    "gcjLongitude": 114.08371000000001,
                    "township": "福田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.078239,
                            22.541019
                        ],
                        [
                            114.076871,
                            22.540687
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 17.918207334026782,
                    "angleDeviationPercent": 9.954559630014879,
                    "distanceDeviationMeter": 4.736462460183759,
                    "distanceDeviationPercent": 3.1576416401225065,
                    "distance": 145.26353753981624,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.076871,
                            22.540687
                        ],
                        [
                            114.078581,
                            22.539935999999997
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 0.34469426157824046,
                    "angleDeviationPercent": 0.19149681198791135,
                    "distanceDeviationMeter": 21.461743888062216,
                    "distanceDeviationPercent": 12.405632305238274,
                    "distance": 194.46174388806222,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.23481899999999,
                        22.722628
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "清林东路84号2层",
                    "name": "皇家缪斯美胸养生馆(龙岗店)",
                    "district": "龙岗区",
                    "gcjLatitude": 22.719796,
                    "gcjLongitude": 114.23966200000001,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.233977,
                        22.721466
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "吉祥路390号",
                    "name": "膳心餐厅(吉祥路)",
                    "district": "龙岗区",
                    "gcjLatitude": 22.718633999999998,
                    "gcjLongitude": 114.23881999999999,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.23555800000001,
                        22.720848999999998
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "龙城街道紫微花园东7栋104室",
                    "name": "深圳足迹文化传播有限公司",
                    "district": "龙岗区",
                    "gcjLatitude": 22.718016,
                    "gcjLongitude": 114.2404,
                    "township": "龙城街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.23481899999999,
                            22.722628
                        ],
                        [
                            114.233977,
                            22.721466
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 22.51284864575942,
                    "angleDeviationPercent": 12.50713813653301,
                    "distanceDeviationMeter": 5.4120792759592575,
                    "distanceDeviationPercent": 3.6080528506395053,
                    "distance": 155.41207927595926,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.233977,
                            22.721466
                        ],
                        [
                            114.23555800000001,
                            22.720848999999998
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 2.736197247720412,
                    "angleDeviationPercent": 1.5201095820668955,
                    "distanceDeviationMeter": 3.0730983386621915,
                    "distanceDeviationPercent": 1.7763574211920183,
                    "distance": 176.0730983386622,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.05904,
                        22.620651000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "光雅园一巷与光雅园五巷交叉口西150米",
                    "name": "天美美容院",
                    "district": "龙岗区",
                    "gcjLatitude": 22.617953,
                    "gcjLongitude": 114.06416499999999,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.057695,
                        22.619891
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "潮州菜",
                    "address": "坂田街道五和社区光雅园八巷18-5号",
                    "name": "潮汕原味汤粉王(新安路)",
                    "district": "龙岗区",
                    "gcjLatitude": 22.617191000000002,
                    "gcjLongitude": 114.06281899999999,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.059577,
                        22.619417000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "坂田雅园路273号手造文化街二期3楼",
                    "name": "时代科腾动画传媒",
                    "district": "龙岗区",
                    "gcjLatitude": 22.616719,
                    "gcjLongitude": 114.06470300000001,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.05904,
                            22.620651000000002
                        ],
                        [
                            114.057695,
                            22.619891
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 2.090716029066158,
                    "angleDeviationPercent": 1.1615089050367544,
                    "distanceDeviationMeter": 11.864623101382534,
                    "distanceDeviationPercent": 7.9097487342550235,
                    "distance": 161.86462310138253,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.057695,
                            22.619891
                        ],
                        [
                            114.059577,
                            22.619417000000002
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 9.918390687697752,
                    "angleDeviationPercent": 5.510217048720973,
                    "distanceDeviationMeter": 27.2331630809079,
                    "distanceDeviationPercent": 15.741712763530577,
                    "distance": 200.2331630809079,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.12800700000001,
                        22.550005
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "凤凰路218号附近",
                    "name": "JIT快速剪发",
                    "district": "罗湖区",
                    "gcjLatitude": 22.5473,
                    "gcjLongitude": 114.13308,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.126576,
                        22.549345000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "中餐厅",
                    "address": "工纺大厦",
                    "name": "朱记小吃馆",
                    "district": "罗湖区",
                    "gcjLatitude": 22.546641,
                    "gcjLongitude": 114.131652,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.128196,
                        22.549318
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "东清平路11号",
                    "name": "长沙勘察院深圳分院",
                    "district": "罗湖区",
                    "gcjLatitude": 22.546612,
                    "gcjLongitude": 114.13326799999999,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.12800700000001,
                            22.550005
                        ],
                        [
                            114.126576,
                            22.549345000000002
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 6.799681715568163,
                    "angleDeviationPercent": 3.7776009530934234,
                    "distanceDeviationMeter": 14.261033563161334,
                    "distanceDeviationPercent": 9.507355708774222,
                    "distance": 164.26103356316133,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.126576,
                            22.549345000000002
                        ],
                        [
                            114.128196,
                            22.549318
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 23.10003535272969,
                    "angleDeviationPercent": 12.833352973738718,
                    "distanceDeviationMeter": 6.608350212495509,
                    "distanceDeviationPercent": 3.8198556141592537,
                    "distance": 166.3916497875045,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.05886799999999,
                        22.620502
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "光雅园步行街1-25",
                    "name": "芳艺美容养生馆",
                    "district": "龙岗区",
                    "gcjLatitude": 22.617803,
                    "gcjLongitude": 114.063993,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.057695,
                        22.619891
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "潮州菜",
                    "address": "坂田街道五和社区光雅园八巷18-5号",
                    "name": "潮汕原味汤粉王(新安路)",
                    "district": "龙岗区",
                    "gcjLatitude": 22.617191000000002,
                    "gcjLongitude": 114.06281899999999,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.059577,
                        22.619417000000002
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "坂田雅园路273号手造文化街二期3楼",
                    "name": "时代科腾动画传媒",
                    "district": "龙岗区",
                    "gcjLatitude": 22.616719,
                    "gcjLongitude": 114.06470300000001,
                    "township": "坂田街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.05886799999999,
                            22.620502
                        ],
                        [
                            114.057695,
                            22.619891
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 4.045182081934083,
                    "angleDeviationPercent": 2.2473233788522684,
                    "distanceDeviationMeter": 11.755200264596596,
                    "distanceDeviationPercent": 7.83680017639773,
                    "distance": 138.2447997354034,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.057695,
                            22.619891
                        ],
                        [
                            114.059577,
                            22.619417000000002
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 9.918390687697752,
                    "angleDeviationPercent": 5.510217048720973,
                    "distanceDeviationMeter": 27.2331630809079,
                    "distanceDeviationPercent": 15.741712763530577,
                    "distance": 200.2331630809079,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    },
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.126779,
                        22.552258
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "美容美发店",
                    "poiMinorType": "美容美发店",
                    "address": "文锦中路1018号海丽大厦裙楼群房1-4楼西侧之一1楼东侧101号",
                    "name": "秀域古方减肥美容保健(凤翔街)",
                    "district": "罗湖区",
                    "gcjLatitude": 22.549554,
                    "gcjLongitude": 114.131854,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.12559399999999,
                        22.551913
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "中餐厅",
                    "poiMinorType": "火锅店",
                    "address": "文锦中路1047-1号",
                    "name": "犇犇牛火锅(罗湖店)",
                    "district": "罗湖区",
                    "gcjLatitude": 22.549211,
                    "gcjLongitude": 114.130672,
                    "township": "东门街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "Point",
                    "coordinates": [
                        114.12694499999999,
                        22.551084
                    ]
                },
                "properties": {
                    "marker-color": "#ff0000",
                    "marker-size": "medium",
                    "poiMajorType": "科教文化场所",
                    "poiMinorType": "科教文化场所",
                    "address": "文锦中路1008号罗湖管理中心大厦12楼1201",
                    "name": "罗湖发展研究中心",
                    "district": "罗湖区",
                    "gcjLatitude": 22.548379999999998,
                    "gcjLongitude": 114.13202,
                    "township": "黄贝街道"
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.126779,
                            22.552258
                        ],
                        [
                            114.12559399999999,
                            22.551913
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 15.327235410291017,
                    "angleDeviationPercent": 8.515130783495009,
                    "distanceDeviationMeter": 22.40626333627273,
                    "distanceDeviationPercent": 14.937508890848486,
                    "distance": 127.59373666372727,
                    "poiIdx1": 0,
                    "poiIdx2": 1
                }
            },
            {
                "type": "Feature",
                "geometry": {
                    "type": "LineString",
                    "coordinates": [
                        [
                            114.12559399999999,
                            22.551913
                        ],
                        [
                            114.12694499999999,
                            22.551084
                        ]
                    ]
                },
                "properties": {
                    "stroke": "#ff0000",
                    "stroke-width": 3,
                    "stroke-opacity": 1,
                    "angleDeviationDegree": 7.479215092194835,
                    "angleDeviationPercent": 4.155119495663797,
                    "distanceDeviationMeter": 6.430423826624491,
                    "distanceDeviationPercent": 3.7170079922684915,
                    "distance": 166.5695761733755,
                    "poiIdx1": 1,
                    "poiIdx2": 2
                }
            }
        ]
    }
]
```
