---
layout:     post
title:      Mybitis Mapper List
subtitle:   Mapper List
date:       2018-08-21
author:     Young
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - Mybitis
    - List
---

>Mybatis 传递List参数调用存储过程匹配表值函数

# 建立四个类，分别是前台传入参数类，参数转换类，（调用存储过程），结果转换类，返回前台结果类
**参数类**
public class CodeBarParam implements ICodeBarParam{
    private Integer baseLine;
    private Double quantity;
    private String itemCode;
    private List<CodeBarItem> codeBarItems;
    get(),set()省略

}

**参数转换类**
public class CodeBarParseParam implements ICodeBarParseParam{
    //传入存储过程参数  匹配表值对象类型
    public static List<CodeBarParseParam> createParseParam(List<CodeBarParam>codeBarParams){
        List<CodeBarParseParam> codeBarParseParams = new ArrayList<>();
        CodeBarParseParam codeBarParseParam;
        for (CodeBarParam codeBarParam:
             codeBarParams) {
            for (CodeBarItem item:
                    codeBarParam.getCodeBarItems()) {
                codeBarParseParam = new CodeBarParseParam();
                codeBarParseParam.setBaseLine(codeBarParam.getBaseLine());
                codeBarParseParam.setItemCode(codeBarParam.getItemCode());
                codeBarParseParam.setQuantity(codeBarParam.getQuantity());
                codeBarParseParam.setCodeBar(item.getCodeBar());
                codeBarParseParam.setCodeBarQty(item.getCodeBarQty());
                codeBarParseParams.add(codeBarParseParam);
            }
        }
        return codeBarParseParams;
    }
    private Integer baseLine;
    private Double quantity;
    private String itemCode;
    private Double codeBarQty;
    private String codeBar;
    get(),set()省略 
}

**结果转换类**
public class CodeBarParseResult implements ICodeBarParseResult {
    private Integer baseLine;
    private String itemCode;
    private String codeBar;
    private Double quantity;
    private Double codeBarQty;
    get(),set()省略   
}

**结果类**
public class CodeBarResult implements ICodeBarResult {

    public static List<CodeBarResult> createCodeBarResult(List<CodeBarParseResult> codeBarParseResults){
        List<CodeBarResult> codeBarResults = new ArrayList<>();
        CodeBarResult codeBarResult ;
        CodeBarItem codeBarItem;
        for (CodeBarParseResult codeBarParseResult:
                codeBarParseResults) {
            codeBarItem = new CodeBarItem();
            codeBarItem.setCodeBar(codeBarParseResult.getCodeBar());
            codeBarItem.setCodeBarQty(codeBarParseResult.getCodeBarQty());

            List<CodeBarResult> newList = codeBarResults.stream()
                    .filter(c->c.getBaseLine().equals(codeBarParseResult.getBaseLine())
                            && c.getItemCode().equals(codeBarParseResult.getItemCode())
                            && c.getQuantity().equals(codeBarParseResult.getQuantity()))
                    .collect(Collectors.toList());

            if(newList != null && newList.size() > 0){
               newList.get(0).getCodeBarItems().add(codeBarItem);
            }else{
                codeBarResult = new CodeBarResult();
                codeBarResult.getCodeBarItems().add(codeBarItem);
                codeBarResult.setItemCode(codeBarParseResult.getItemCode());
                codeBarResult.setBaseLine(codeBarParseResult.getBaseLine());
                codeBarResult.setQuantity(codeBarParseResult.getQuantity());
                codeBarResults.add(codeBarResult);
            }
        }
        return codeBarResults;
    }

    public CodeBarResult(){
        this.codeBarItems = new ArrayList<>();
    }
    private Integer baseLine;
    private String itemCode;
    private Double quantity;
    private List<CodeBarItem> codeBarItems;
     get(),set()省略 
}




# 由于jdbcType没有List类型，所以需要自定义一个TableTypeHandler
**直接上代码，字段根据需要进行转换**
@MappedJdbcTypes(JdbcType.ARRAY)
public class TableTypeHandler extends BaseTypeHandler<Object> {

    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, Object parameter, JdbcType jdbcType) throws SQLException {
        //从代码中可以看出进行转换的是  CodeBarParseParam（参数转换类）即需要调用存储过程传递的List泛型对象
        SQLServerDataTable sourceDataTable = new SQLServerDataTable();
        sourceDataTable.addColumnMetadata("CodeBar" , Types.NVARCHAR);
        sourceDataTable.addColumnMetadata("BaseLine" , Types.INTEGER);
        sourceDataTable.addColumnMetadata("ItemCode" , Types.NVARCHAR);
        sourceDataTable.addColumnMetadata("Quantity" , Types.DOUBLE);
        sourceDataTable.addColumnMetadata("CodeBarQty" , Types.DOUBLE);
        List<CodeBarParseParam> dataList = (List)parameter;
        for (CodeBarParseParam item : dataList) {
             sourceDataTable.addRow(item.getCodeBar(),item.getBaseLine(),item.getItemCode(),item.getQuantity(),item.getCodeBarQty());
        }
        ps.setObject(i, sourceDataTable);
    }

    @Override
    public Object getNullableResult(ResultSet rs, String columnName) throws SQLException {

        return null;
    }

    @Override
    public Object getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        return null;
    }

    @Override
    public Object getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        return null;
    }
}


# TableTypeHandler建立完毕后，需要配置XML文件，引入自定义参数类型

**代码**
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >
<!-- <mapper namespace="org.edi.stocktask.mapper.CodeBarMapper"> -->
    <resultMap id="BaseResultMap" type="org.edi.stocktask.bo.codeBar.CodeBar">
        <result column="ProName" property="proName" jdbcType="VARCHAR" />
        <result column="ProValue" property="proValue" jdbcType="VARCHAR" />
        <result column="ProDesc" property="proDesc" jdbcType="VARCHAR" />
    </resultMap>

    <parameterMap id="codeBarParam" type="map">
        <parameter property="itemCode" jdbcType="NVARCHAR" mode="IN"/>
        <parameter property="codebar" jdbcType="NVARCHAR" mode="IN"/>
        <parameter property="baseType" jdbcType="NVARCHAR" mode="IN"/>
        <parameter property="baseEntry" jdbcType="INTEGER" mode="IN"/>
        <parameter property="baseLine" jdbcType="INTEGER" mode="IN"/>
    </parameterMap>


    <!-- 此为调用存储过程传递List<Object>的具体配置-->
    <parameterMap id="BatchCodeBarParamMap" type="map">
        <parameter property="codeBarParams" jdbcType="ARRAY" javaType="java.util.List" mode="IN" typeHandler="org.edi.stocktask.util.TableTypeHandler"/>
        <parameter property="baseType" jdbcType="VARCHAR" mode="IN"/>
        <parameter property="baseEntry" jdbcType="INTEGER" mode="IN"/>
    </parameterMap>


    <!-- 此为调用存储过程传递List<Object>的具体配置-->
    <resultMap id="BatchCodeBarResultMap" type="org.edi.stocktask.dto.CodeBarParseResult">
        <result column="BaseType" property="baseType" jdbcType="VARCHAR" />
        <result column="BaseEntry" property="baseEntry" jdbcType="INTEGER" />
        <result column="BaseLine" property="baseLine" jdbcType="INTEGER" />
        <result column="ItemCode" property="itemCode" jdbcType="VARCHAR" />
        <result column="CodeBar" property="codeBar" jdbcType="VARCHAR" />
    </resultMap>


    <select id="parseCodeBar" statementType="CALLABLE" parameterMap="codeBarParam" resultMap="BaseResultMap">
        {call AVA_SP_PARSE_CODEBAR(
        #{itemCode, jdbcType = NVARCHAR, mode = IN},
        #{codebar, jdbcType = NVARCHAR, mode = IN},
        #{baseType, jdbcType = NVARCHAR, mode = IN},
        #{baseEntry, jdbcType = INTEGER, mode = IN},
        #{baseLine, jdbcType = INTEGER, mode = IN},
        #{code, jdbcType = INTEGER, mode = OUT},
        #{message, jdbcType = NVARCHAR, mode = OUT}
        )}
    </select>

    <!-- 此为调用存储过程传递List<Object>的具体配置-->
    <!-- List<CodeBarAnalysis> parseBatchCodeBar(HashMap<String,Object> codeBarParamsList);-->
    <select id="parseBatchCodeBar" statementType="CALLABLE" parameterMap="BatchCodeBarParamMap"  resultMap="BatchCodeBarResultMap">
        {call AVA_SP_BATCH_PARSE_CODEBAR(
        #{codeBarParams,typeHandler = org.edi.stocktask.util.TableTypeHandler,jdbcType = ARRAY, mode = IN},
        #{baseType, jdbcType = NVARCHAR, mode = IN},
        #{baseEntry, jdbcType = INTEGER, mode = IN},
        #{code, jdbcType = INTEGER, mode = OUT},
        #{message, jdbcType = NVARCHAR, mode = OUT}
        )}
    </select>
<!-- </mapper> -->




# 注：先由CodeBarParam接收前台传递参数，转换为CodeBarParseParam，用List<CodeBarParseParam>作为输入参数调用存储过程，调用完毕后，以CodeBarParseResult作为ResultMap去接收，接收之后再以CodeBarResult转换为规定格式，返回给前台



