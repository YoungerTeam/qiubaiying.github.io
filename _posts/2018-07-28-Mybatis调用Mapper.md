---
layout:     post
title:      Mybitis Mapper
subtitle:   Mapper
date:       2018-07-28
author:     Young
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mybitis
    - Mapper
---

>Mybatis 引用映射文件

# 配置mybatis-config.xml文件

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC" />
            <!-- 配置数据库连接信息 -->
            <dataSource type="POOLED">
                <property name="driver" value="com.microsoft.sqlserver.jdbc.SQLServerDriver"/>
                <property name="url" value="jdbc:sqlserver://192.168.3.14;DatabaseName=StockManageDB"/>
                <property name="username" value="sa"/>
                <property name="password" value="avatech"/>
            </dataSource>
        </environment>
    </environments>
    <!-- 配置此项目模块中的Mapper -->
    <mappers>
        <mapper resource="mappers/StockTaskMapper.xml"/>
        <mapper resource="mappers/StockReportMapper.xml"/>
        <mapper resource="mappers/CodeBarMapper.xml"/>
    </mappers>

</configuration>

# 编写单元测试类引入两个方法
**打开SqlSession**
public SqlSession getSqlSession() throws IOException {
        String resource = "mybatis-config.xml";
        InputStream inputStream = Resources.getResourceAsStream(resource);
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession openSession = sqlSessionFactory.openSession(true);
        return openSession;
    }

**加载映射接口StockTaskMapper.class（根据需要改变）**
    public StockTaskMapper getStockTaskMapper() throws IOException {
        StockTaskMapper stockTaskMapper=getSqlSession().getMapper(StockTaskMapper.class);
        return stockTaskMapper;
    }


# 引入业务类中要测试的方法

**将getStockTaskMapper()替换掉原autowired接口注入mapper**
/**
     * 查询库存任务
     * @return
     */
    public List<StockTask> fetchStockTask(String param)throws IOException{
        List<StockTask> stockTasks;
        if(param!=null && !param.isEmpty()){
            stockTasks = getStockTaskMapper().fetchStockTaskFuzzy(param);
        }else {
            stockTasks = getStockTaskMapper().fetchStockTask();
        }
        if(stockTasks.size() == 0) {
            return stockTasks;
        }
        for (int i = 0;i<stockTasks.size();i++){
            List<StockTaskItem> stockTaskItems = getStockTaskMapper().fetchStockTaskItem(stockTasks.get(i).getObjectKey());
            if(stockTaskItems!=null){
                stockTasks.get(i).setStockTaskItems(stockTaskItems);
            }
        }
        return stockTasks;
    }

# JUIT单元测试
@Test
    public void fetchStockTaskTest() throws IOException {
        List<StockTask> stockTasks = fetchStockTask(null);
            for (StockTask stockTask:stockTasks) {
                System.out.println(stockTask.toString());
            }
    }


