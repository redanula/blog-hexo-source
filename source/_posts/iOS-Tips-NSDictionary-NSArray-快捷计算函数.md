title: "iOS Tips--NSDictionary和NSArray 快捷计算函数"
date: 2017-07-30 23:15:46
tags: [iOS,KVC]
---
NSArray 有快捷的汇总公式，只要用 @函数.键路径 即可：

        NSArray *array = [NSArray arrayWithObjects:@"1.0",@"2.0",nil];
        CGFloat sum = [[array valueForKeyPath:@"@sum.floatValue"] floatValue];
        CGFloat avg = [[array valueForKeyPath:@"@avg.floatValue"] floatValue];
        CGFloat max =[[array valueForKeyPath:@"@max.floatValue"] floatValue];
        CGFloat min =[[array valueForKeyPath:@"@min.floatValue"] floatValue];

如果汇总值是在NSDictionary里面，同样可以取值进行汇总：
    
        NSArray *dic = [{@"KeyName1":@"1.0",@"KeyName2":@"abc"},{@"KeyName1":@"2.0",@"KeyName2":@"bcd"}]; 
        CGFloat sum = [[[dic valueForKeyPath:@"KeyName1"] valueForKeyPath:@"@sum.floatValue"] floatValue];
        CGFloat max = [[[dic valueForKeyPath:@"KeyName1"] valueForKeyPath:@"@max.floatValue"] floatValue];
        
以上快捷汇总实际是KVC的用法，更多KVC的用法参考
1.<http://www.jianshu.com/p/b7dda8d49dc4>
2.<https://developer.apple.com/library/content/documentation/General/Conceptual/DevPedia-CocoaCore/KeyValueCoding.html>

