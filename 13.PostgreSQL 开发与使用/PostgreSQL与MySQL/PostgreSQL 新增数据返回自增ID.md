- [PostgreSQL 新增数据返回自增ID_张志翔 ̮的博客-CSDN博客_postgresql 获取自增id](https://vegetable-chicken.blog.csdn.net/article/details/104901750)

最近在项目中需要[Postgresql](https://so.csdn.net/so/search?q=Postgresql&spm=1001.2101.3001.7020)在新增数据后返回自增的ID，特此记录便于日后查阅。

```plsql
<insert id="copyMainGroup" parameterType="com.openailab.oascloud.common.model.tcm.TrainingGroupBO">
    <selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">
        SELECT currval('training_group_id_seq'::regclass) AS id
    </selectKey>
    INSERT INTO t_teaching_training_group(
    outline_id,
    total_student,
    max_count,
    group_no,
    parent_id,
    status,
    type,
    min_gpu_cores,
    gpu_specs,
    min_cpu_cores,
    min_memory,
    min_store,
    plan_training_start_date,
    plan_training_end_date,
    create_date,
    modify_date,
    create_user,
    is_delete,
    issue_training_date
    )
    VALUES
    (
    #{outlineId},
    #{totalStudent},
    #{maxCount},
    #{groupNo},
    #{id},
    #{status},
    #{type},
    #{minGpuCores},
    #{gpuSpecs},
    #{minCpuCores},
    #{minMemory},
    #{minStore},
    #{retrainingStartDate},
    #{retrainingEndDate},
    now(),
    now(),
    #{createUser},
    #{isDelete},
    #{issueTrainingDate}
    )
</insert>
```

上述代码重点关注这一代码段，代码如下：

```plsql
<selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">
    SELECT currval('training_group_id_seq'::regclass) AS id
</selectKey>
```

上述代码的作用是，在数据插入完成后返回主键ID。

**注意：这里有个坑，keyProperty="id，这里指的是将主键id返回给传入实体的哪一个字段中，这里就是自增的主键id会放到传入TrainingGroupBO中的id属性中，如果要获取自增的id的话通过 trainingGroupBO.getId() 获取（我在这里吃了亏，以为Mybatis INSERT返回值就是自增后的id，结果调试了一个小时不管怎么修改返回的都是1）。**

到此 Postgresql 插入数据返回自增ID介绍完成。