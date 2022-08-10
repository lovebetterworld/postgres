- [pgr_createVerticesTable — pgRouting Manual (2.2)](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_createVerticesTable.html)

基于源和目标信息重新构建顶点表。

```sql
varchar pgr_createVerticesTable(text edge_table,  text the_geom:='the_geom'
                                text source:='source',text target:='target',text rows_where:='true')
```

顶点表函数的重建接受以下参数:

| edge_table: | `text` 表名 (may contain the schema name as well)            |
| :---------- | ------------------------------------------------------------ |
| the_geom:   | `text` Geometry column name of the network table. Default value is `the_geom`. |
| source:     | `text` Source column name of the network table. Default value is `source`. |
| target:     | `text` Target column name of the network table. Default value is `target`. |
| rows_where: | `text` Condition to SELECT a subset or rows. Default value is `true` to indicate all rows. |

The `edge_table` will be affected

> - An index will be created, if it doesn’t exists, to speed up the process to the following columns:
>
>   > - `the_geom`
>   > - `source`
>   > - `target`

The function returns:

成功：

- after the vertices table has been reconstructed.

- Creates a vertices table: <edge_table>_vertices_pgr.

- Fills `id` and `the_geom` columns of the vertices table based on the source and target columns of the edge table.

失败：

- when the vertices table was not reconstructed due to an error.

- A required column of the Network table is not found or is not of the appropriate type.

- The condition is not well formed.

- The names of source, target are the same.

- The SRID of the geometry could not be determined.

顶点表的结构是：

| id:       | `bigint` Identifier of the vertex.                           |
| :-------- | ------------------------------------------------------------ |
| cnt:      | `integer` Number of vertices in the edge_table that reference this vertex. See [*pgr_analyzeGraph*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeGraph.html#pgr-analyze-graph). |
| chk:      | `integer` Indicator that the vertex might have a problem. See [*pgr_analyzeGraph*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeGraph.html#pgr-analyze-graph). |
| ein:      | `integer` Number of vertices in the edge_table that reference this vertex as incoming. See [*pgr_analyzeOneway*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeOneWay.html#pgr-analyze-oneway). |
| eout:     | `integer` Number of vertices in the edge_table that reference this vertex as outgoing. See [*pgr_analyzeOneway*](https://docs.pgrouting.org/2.2/en/src/topology/doc/pgr_analyzeOneWay.html#pgr-analyze-oneway). |
| the_geom: | `geometry` Point geometry of the vertex.                     |