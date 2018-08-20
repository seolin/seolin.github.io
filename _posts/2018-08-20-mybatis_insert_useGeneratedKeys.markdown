---
layout:     post
title:      "[转]mybatis返回自增主键"
subtitle:   " \" mybatis返回自增主键的结论与建议的写法 \""
date:       2018-08-16
author:     "Aziz"
header-img: "img/in-post/mybatis_insert_useGenerateKeys/mybatis_insert_useGenerateKeys.png"
tags:
    - mybatis
    - database
---

> “mybatis返回的自增主键”


## 前言

本文仅仅记录返回主键的结论，不进行源码分析

## 结论
以下insertOrUpdate 为insert on duplicate key update
+ 使用@Param注入入参时，在任何情况下keyProperty必须用"@Param的入参名字.主键属性名" (entity.id)的形式才能正确解析到入参对象上并设置主键
+ 批量insert时，采用@Param的注解是没有办法返回主键的，所以paramenterType在返回主键上比@Param更具有通用性。
+ insertOrUpdate时，selectKey只能正确返回插入时候的主键，无法正确返回更新时候的主键，useGenerateKeys在插入或者更新的时候都能够正确的返回主键。所以在insertOrUpdate时采用useGeneratedKey。
+ 批量insertOrUpdate时候，任何情况都不能正确的返回主键，所以程序逻辑不能依赖返回的主键。

## 实践

+ 插入

    ```
    int insert(BoyEntity entity);
    
    <insert id="insert" useGeneratedKeys="true" keyProperty="id" parameterType="BoyEntity">
            insert into boy (`name`, created_time, modified_time)
            values (#{name}, now(), now())
    </insert>
    ```
+ 批量插入
    
    ```
    int insertBatch(List<BoyEntity> entityList);
    
    <insert id="insertBatch" useGeneratedKeys="true" keyProperty="id">
            insert into boy (`name`, created_time, modified_time)
            values 
            <foreach collection="list" item="entity" index= "index" separator =",">
                (#{entity.name}, now(), now())
            </foreach>
    </insert>
    ```
+ 插入或更新
    
    ```
    int insertOrUpdate(BoyEntity entity);
    
    <insert id="insertOrUpdateUseGeneratedKeys" useGeneratedKeys="true" keyProperty="id" parameterType="BoyEntity">
            insert into boy (`name`, created_time, modified_time)
            values (#{name}, now(), now())
            on duplicate key update modified_time = now()
    </insert>
    ```
+ 批量插入或更新

    ```
    int insertOrUpdateBatch(List<BoyEntity> entityList);
    
    <insert id="insertOrUpdateBatch">
            insert into boy (`name`, created_time, modified_time)
            values 
            <foreach collection="list" item="entity" index= "index" separator =",">
                (#{entity.name}, now(), now())
            </foreach>
            on duplicate key update modified_time = now()
        </insert>
    
    ```

## 参考
+ [避坑必看：很详尽的MyBatis返回自增主键实验（包括插入或更新SQL语句insert on duplicate key update的自增主键返回情况）](https://blog.csdn.net/qq_27680317/article/details/81118070)
