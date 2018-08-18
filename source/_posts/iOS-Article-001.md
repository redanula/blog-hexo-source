title: iOS本地时间与同步
date: 2017-07-31 21:52:35
tags:
---

### 概念

GMT（Greenwich Mean Time）格林尼治时间
UTC（Coordinated Universal Time ）原子钟标准时间
Unix time（以UTC 1970年1月1号 00：00：00为基准时间）

---

### sysctl API

iOS系统上次设备重启时间（iOS9后不能使用）

    #include <sys/sysctl.h>

    - (long)bootTime
    {
        #define MIB_SIZE 2
        int mib[MIB_SIZE];
        size_t size;
        struct timeval  boottime;
        
        mib[0] = CTL_KERN;
        mib[1] = KERN_BOOTTIME;
        size = sizeof(boottime);
        if (sysctl(mib, MIB_SIZE, &boottime, &size, NULL, 0) != -1)
        {
            return boottime.tv_sec;
        }
        return 0;
    }
    
### 时间校准



1、记录上一次请求的时候获取服务器时间serverTime，与本地时间lastLocalTime

2、需要计算的时候取出curLocalTime（即NSDate），计算与lastLocalTime的偏移量T

3、serverTime+T 就是当前的服务器时间



##### 另外，计算运行时长，计算自上一次重启后的运行时间
    //get system uptime since last boot
    - (NSTimeInterval)uptime
    {
        struct timeval boottime;
        int mib[2] = {CTL_KERN, KERN_BOOTTIME};
        size_t size = sizeof(boottime);
    
        struct timeval now;
        struct timezone tz;
        gettimeofday(&now, &tz);
        
        double uptime = -1;
        
        if (sysctl(mib, 2, &boottime, &size, NULL, 0) != -1 && boottime.tv_sec != 0)
        {
            uptime = now.tv_sec - boottime.tv_sec;
            uptime += (double)(now.tv_usec - boottime.tv_usec) / 1000000.0;
        }
        return uptime;
    }

详细参考 <http://mrpeak.cn/blog/ios-time/>

