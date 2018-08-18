title: NSDictionary NSMutableDictionary
date: 2015-08-18 21:35:24
tags: [NSDictionary,NSMutableDictionary]
---

转自：<http://seven-sally.lofter.com/post/19d861_5404fa>

## 不可变词典NSDictionary

### 字典初始化

    NSNumber *numObj = [NSNumber numberWithInt:100];

### 以一个元素初始化

    NSDictionary *dic = [NSDictionary dictionaryWithObject:numObj forKey:@"key"];

### 初始化两个元素

    NSDictionary *dic = [NSDictionary dictionaryWithObjectsAndKeys:numObj, @"valueKey", numObj2, @"value2",nil];

### 初始化新字典，新字典包含otherDic

    NSDictionary *dic = [NSDictionary dictionaryWithDictionary:otherDic];

### 以文件内容初始化字典

    NSDictionary *dic = [NSDictionary dictionaryWithContentsOfFile:path];


### 常用方法

### 获取字典数量

    NSInteger count = [dic count];

### 通过key获取对应的value对象

    NSObject *valueObj = [dic objectForKey:@"key"];

### 将字典的key转成枚举对象，用于遍历

    NSEnumerator *enumerator = [dic keyEnumerator];

### 获取所有键的集合

    NSArray *keys = [dic allKeys];

### 获取所有值的集合

    NSArray *values = [dic allValues];


</br>

## 可变数组NSMutableDictionary

### 初始化一个空的可变字典

    NSMutableDictionary *dic2 = [NSMutableDictionary dictionaryWithObjectsAndKeys:@"v1",@"key1",@"v2",@"key2",nil];

    NSDictionary *dic3 = [NSDictionary dictionaryWithObject:@"v3" forKey:@"key3"];

### 向字典2对象中添加整个字典对象3

    [dic2 addEntriesFromDictionary:dic3];

### 向字典2对象中最佳一个新的key3和value3

    [dic2 setValue:@"value3" forKey:@"key3"];

### 初始化一个空的可变字典

    NSMutableDictionary *dic1 = [NSMutableDictionary dictionary];

### 将空字典1对象内容设置与字典2对象相同

    [dic1 setDictionary:dic2];

### 将字典中key1对应的值删除

    [dic1 removeObjectForKey@"key1"];

    NSArray *array = [NSArray arrayWithObjects:@"key1", nil];

### 根据指定的数组（key）移除字典1的内容

[dic2 removeObjectsForKeys:array];

### 移除字典所有对象

    [dic1 removeAllObjects];


### 遍历字典

### 快速枚举

    for (id key in dic){

        id obj = [dic objectForKey:key];

        NSLog(@"%@", obj);

    }

### 一般枚举

    NSArray *keys = [dic allKeys];

    inr length = [keys count];

    for (int i = 0; i < length；i++){

        id key = [keys objectAtIndex:i];

        id obj = [dic objectForKey:key];

        NSLog(@"%@", obj);

    }

### 通过枚举类型枚举

    NSEnumerator *enumerator = [dic keyEnumerator];
     
    id key = [enumerator nextObject];
    
    while (key) {
    
        id obj = [dic objectForKey:key];
        
        NSLog(@"%@", obj);
        
        key = [enumerator nextObject];
    }

