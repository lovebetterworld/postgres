[使用pgrouting和geotools实现最短路径，服务区分析_玛卡瑞那的博客-CSDN博客_dijkstra geotools](https://blog.csdn.net/zhgu1992/article/details/78883993)

## 1 本文主要讲解服务区分析的实现

（最优路径已经有很多文章了）

设施服务范围指在一定限制条件下(如时间、费用或路程等)设施所能提供服务的最大空间领域, 在道路网络环境中,它通常由一系列结点及边组成。例如, 某救助站在接到求救电话后10 min 所能到达的区域;某物流公司在配送货物时500元花费所能到达的区域等。

（1）根据拓扑关系,计算地理网络的最大邻接结点数;

（2）构造邻接结点[矩阵](https://so.csdn.net/so/search?q=矩阵&spm=1001.2101.3001.7020)和初始判断矩阵描述地理网络结构;

（3）应用广度优先搜索算法确定地理网络中心服务范围。

本算法是对Dijkstra[最短路径](https://so.csdn.net/so/search?q=最短路径&spm=1001.2101.3001.7020)算法的改进(简称“最短路径算法”)。首先, 将网络中所有结点初始化为未标记结点。然后从起点(第一次搜索的起点为网络中心)开始搜索与其有路径连通的未标记结点, 计算阻值, 并将起点标记为已标记结点, 重复上述过程, 直到某结点的阻值超过网络中心的阻值。最后, 基于结点及边的阻值搜索并存储所有在中心阻值范围内的边, 这些边和结点的集合为网络中心的服务范围。

（但实际情况中可能需要内插一些点，直到找到阻值等于网络中心的阻值为止）

## 2 实现过程

### 2.1 数据读取：直接读取shp

```java
//1读取shp文件，得到pgDatastore
public static void conShp(String path){
    try {
        File file=new File(path);
        Map<String, Object> map = new HashMap<String, Object>();
        map.put("url", file.toURI().toURL());
        System.out.println(map);
        pgDatastore = DataStoreFinder.getDataStore(map);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

 从postgis中读取

首先读取postgis数据库得到DataStore对象，然后用getfeaturesource（LayerName）得到SimpleFeatureSource即可（注意：这里的LayerName即为表名）

```java
//2读取postgis，得到pgDatastore
//链接postgis
public static void conPostGis(String dbtype, String host, String port, 
                              String database, String userName, String password) { 
    Map<String, Object> params = new HashMap<String, Object>(); 
    params.put(PostgisNGDataStoreFactory.DBTYPE.key, dbtype); 
    params.put(PostgisNGDataStoreFactory.HOST.key, host); 
    params.put(PostgisNGDataStoreFactory.PORT.key, new Integer(port)); 
    params.put(PostgisNGDataStoreFactory.DATABASE.key, database); 
    params.put(PostgisNGDataStoreFactory.SCHEMA.key, "public"); 
    params.put(PostgisNGDataStoreFactory.USER.key, userName); 
    params.put(PostgisNGDataStoreFactory.PASSWD.key, password); 
    try { 
        pgDatastore = DataStoreFinder.getDataStore(params); 
        if (pgDatastore != null) { 
            System.out.println("系统连接到位于：" + host + "的空间数据库" + database 
                               + "成功！"); 
        } else { 
            System.out.println("系统连接到位于：" + host + "的空间数据库" + database 
                               + "失败！请检查相关参数"); 
        } 
    } catch (IOException e) { 
        e.printStackTrace(); 
        System.out.println("系统连接到位于：" + host + "的空间数据库" + database 
                           + "失败！请检查相关参数"); 
    } 
}
```

```java
//3利用pgDatastore，得到featuresource(表)
public static SimpleFeatureSource getFeatureSource(String LayerName) throws IOException{
    if(pgDatastore==null){
        System.out.println("还未导入数据源，请导入pgDatastore");
        return null;
    }
    featureSource = pgDatastore.getFeatureSource(LayerName);
    System.out.println(featureSource.getCount(Query.ALL));
    return featureSource;
}
```

**注意事项：**

读取postgis时，数据库里面的geom字段不能为二进制

读取文件时，文件中最好不要有中文

### 2.2 进行拓扑将数据处理为Graph

（1）得到SimpleFeatureCollection

（2）创建一个FeatureGraphGenerator利用它添加SimpleFeature元素并调用其getGraph方法创建Graph

（3）创建出来的Graph中保存着V（节点）和E（边），这样就可以进行网络分析了

```java
//创建graph
public static Graph getGraph(SimpleFeatureSource source) throws IOException{
    if(source==null)
    {   
        System.out.println("资源不存在，请先得到featureSource");
        return null;
    }
    SimpleFeatureCollection fCollection = source.getFeatures();
    //create a linear graph generate
    //构图时也可以创建一个DirectedLineStringGraphGenerator构建有向图
    LineStringGraphGenerator lineStringGen = new LineStringGraphGenerator();
    //wrap it in a feature graph generator
    FeatureGraphGenerator featureGen = new FeatureGraphGenerator( lineStringGen );
    //throw all the features into the graph generator
    FeatureIterator<SimpleFeature> iter = fCollection.features();
    try {
        while(iter.hasNext()){
            Feature feature = iter.next();
            featureGen.add(feature);
        }
    } finally {
        iter.close();
    }
    graph = featureGen.getGraph();
    return graph;
}
```

## 3 最短路径

### 3.1 使用Astar算法

1.首先利用AstarFunctions设定权值(即通过此边的消耗)

2.然后设定start点（起点）和target点（终点）

3.调用AstarShortestFinder（）来进行处理

具体代码如下：

设定权（成本）:

```java
public static double discost(Edge e ){
    SimpleFeature feature = (SimpleFeature) e.getObject();
    Geometry geom = (Geometry) feature.getDefaultGeometry();
    //geom.convexHull()将其构成一个图形
    if(Barriers!=null){
        for(int i=0;i<Barriers.size();i++){
            Geometry g=Barriers.get(i);
            if(geom.intersects(g)){
                return Double.POSITIVE_INFINITY;
            }
        }
    }
    return geom.getLength();
}

public static double discost(AStarNode n1, AStarNode n2){
    Node nd1=n1.getNode();
    Node nd2=n2.getNode();
    Edge e=nd1.getEdge(nd2);
    if(e!=null){
        SimpleFeature feature=(SimpleFeature)e.getObject();
        Geometry geom=(Geometry) feature.getDefaultGeometry();
        if(Barriers!=null){
            for(int i=0;i<Barriers.size();i++){
                Geometry g=Barriers.get(i);
                if(geom.intersects(g)){
                    return Double.POSITIVE_INFINITY;
                }
            }
        }
        return ((Point) n1.getNode().getObject())
            .distance((Point)n2.getNode().getObject());

    }else{
        return ((Point) n1.getNode().getObject())
            .distance((Point)n2.getNode().getObject());
    }
}
```

```java
//Astar方法的最短路径计算
public static Path searchRouteByAstar(Node source,Node destination) throws Exception{
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    if(source.equals(destination)){
        System.out.println("起点和终点相同，请重新选点");
        return null;
    }
    Path path=null;
    AStarFunctions afuncs=new AStarFunctions(destination) {
        @Override
        public double h(Node n) {
            //结合Astar的算法可以知道这里的h指的是一个预估的距离destination的消耗值
            //disPoint指的是预估的终点
            Point disPoint=(Point)this.getDest().getObject();
            return ((Point)n.getObject()).distance(disPoint);		
        }

        @Override
        public double cost(AStarNode n1, AStarNode n2) {
            //注意矢量性和有向性
            return discost(n1, n2);
        }
    };

    AStarShortestPathFinder finder=new AStarShortestPathFinder(graph, source, destination, afuncs);
    finder.calculate();
    path=finder.getPath();
    return path;
}
```

这里是可以看到传入的变量是node节点，但是我们实际中是要在地图上点击一个起点终点求出最优路径，因此还需要将鼠标点击的任意一点归算的graph的节点里去，这里最好使用数据库空间查询来算，本文只是用了最简单的遍历，算法如下：

```java
//搜寻graph上最近节点的方法
//暂时先采用遍历的方法
//这里如果点隔的太远会直接把pointy输出，调用最短路径算法会抛出空指针异常
public static Node getNearestGraphNode(Point pointy){
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    double dist=0;
    Node nearestNode=null;
    for(Object o:graph.getNodes()){
        Node n=(Node)o;
        Point gPoint=(Point)n.getObject();
        double distance=gPoint.distance(pointy);	
        if(nearestNode==null||distance<dist){
            dist=distance;
            nearestNode=n;
        }
    }
    return nearestNode;
}
```

归算到节点之后就可以改造下Astar算法了：

```java
public static Path searchRouteByAstar(Point startPoint,Point endPoint) throws Exception{
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    Node source=getNearestGraphNode(startPoint);
    Node destination=getNearestGraphNode(endPoint);
    if(source.equals(destination)){
        System.out.println("起点和终点相同，请重新选点");
        return null;
    }
    Path path=null;
    AStarFunctions afuncs=new AStarFunctions(destination) {
        @Override
        public double h(Node n) {
            //结合Astar的算法可以知道这里的h指的是一个预估的距离destination的消耗值
            //disPoint指的是预估的终点
            Point disPoint=(Point)this.getDest().getObject();
            return ((Point)n.getObject()).distance(disPoint);		
        }

        @Override
        public double cost(AStarNode n1, AStarNode n2) {
            //注意矢量性和有向性
            return discost(n1, n2);
        }
    };

    AStarShortestPathFinder finder=new AStarShortestPathFinder(graph, source, destination, afuncs);
    finder.calculate();
    path=finder.getPath();
    return path;
}
```

这样看起来就挺完美了，但是如果要加入障碍点怎么办那？

其实我们在成本计算中已经考虑障碍物了，如果是个障碍范围就与当前的graph求交集，交集处的权设置成无穷就好了，这样就解决了障碍点的问题。

如果是停靠点那？

那就每段都计算一次最优路径加起来就行了。

### 3.2 使用Dijkstra算法

1.首先利用Edgeweighter设定权值(即通过此边的消耗)

2.然后设定start点（起点）和target点（终点）

3.调用DijkstraShortestPathFinder（）来进行处理

dijkstra算法大概差不多，直接贴代码：

```java
//dijkstra方法
public static Path searchRouteByDijkstra(Node source,Node destination) throws Exception{
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    Path path=null;
    EdgeWeighter weighter = new EdgeWeighter() {
        @Override
        public double getWeight(Edge e) {
            return discost(e);
        }
    };
    // Create GraphWalker - in this case DijkstraShortestPathFinder
    DijkstraShortestPathFinder pf = new DijkstraShortestPathFinder( graph, source, weighter );
    pf.calculate();
    path= pf.getPath(destination);		   
    return path;
}

public static Path searchRouteByDijkstra(Point startPoint,Point endPoint) throws Exception{
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    Node source=getNearestGraphNode(startPoint);
    Node destination=getNearestGraphNode(endPoint);
    Path path=null;
    EdgeWeighter weighter = new EdgeWeighter() {
        @Override
        public double getWeight(Edge e) {
            return discost(e);
        }
    };
    // Create GraphWalker - in this case DijkstraShortestPathFinder
    DijkstraShortestPathFinder pf = new DijkstraShortestPathFinder( graph, source, weighter );
    pf.calculate();
    path= pf.getPath(destination);		   
    return path;
}
```

## 4 服务区分析

改造DijkstraShortestPathFinder方法：

1.首先通过Edgeweighter设定权值(即通过此边的消耗)

2.然后设定start点（起点）

3.最后通过设置一个判定（该判定可能是根据距离也可能是根据时间）来终止该方法的搜索，然后得到该方法返回的所有边和节点。

```java
public static List<Point> getAdjancyPoint(Node node){
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return null;
    }
    List<Point> points=new ArrayList<Point>(); 
    Point pt=(Point)node.getObject();
    System.out.println("传入的节点："+pt);
    List<Edge> edges=node.getEdges();
    for(Edge e:edges){
        Node nodeA=e.getNodeA();
        Point pa=(Point)nodeA.getObject();
        Node nodeB=e.getNodeB();
        Point pb=(Point)nodeB.getObject();
        if(!pt.equals(pa)){
            points.add(pa);
        }else if(!pt.equals(pb)){
            points.add(pb);
        }
    }
    List<Point>points1=(List<Point>) CollectionUtils.subtract(points,serviceAreaPoints);
    System.out.println("加入的临近点："+points1);
    return points1;
}

//服务区范围,目前我只是把节点加入进去
public static void ServiceArea(Point startPoint, double cost) throws Exception{
    if(graph==null){
        System.out.println("graph不存在，请构建graph");
        return;
    }
    Node source=getNearestGraphNode(startPoint);
    Point pt=(Point)source.getObject();
    serviceAreaPoints.add(pt);
    //其实递归应该从这里开始，前面的不用递归
    List<Point> pts=getAdjancyPoint(source);
    for(Iterator<?>itr=pts.iterator();itr.hasNext();){
        Point p=(Point)itr.next();	
        if(p!=null){
            Geometry geo=iterRoute(searchRouteByAstar(serviceAreaPoints.get(0), p)).getRoutePath();
            double len=geo.getLength();
            if(len<=cost){
                ServiceArea(p, cost);
                System.out.println("点"+p+"加人serviceArea");
            }
            else{
                System.out.println("点"+p+"不加人serviceArea");
            }
        }
    }
}

//获得服务区点集合
public static Set<Point> getServiceAreaPoints() {
    serviceAreaPoints1.clear();
    serviceAreaPoints1.addAll(serviceAreaPoints);
    return serviceAreaPoints1;
}
```