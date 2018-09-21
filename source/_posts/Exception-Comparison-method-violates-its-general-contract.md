---
title: Exception - Comparison method violates its general contract
date: 2018-09-21 18:29:31
categories:
 - technology
tags:
 - technology
 - notes
 - exception
---


##### 在测试服务器上出现错误
```shell

java.lang.IllegalArgumentException: Comparison method violates its general contract!
        at java.util.TimSort.mergeHi(TimSort.java:899) ~[?:1.8.0_131]
        at java.util.TimSort.mergeAt(TimSort.java:516) ~[?:1.8.0_131]
        at java.util.TimSort.mergeCollapse(TimSort.java:439) ~[?:1.8.0_131]
        at java.util.TimSort.sort(TimSort.java:245) ~[?:1.8.0_131]
        at java.util.Arrays.sort(Arrays.java:1512) ~[?:1.8.0_131]
        at java.util.ArrayList.sort(ArrayList.java:1454) ~[?:1.8.0_131]
        at java.util.Collections.sort(Collections.java:175) ~[?:1.8.0_131]
        at com.mixky.asp.commons.design.util.DesignObjectUtil.sortListWithOrderField(DesignObjectUtil.java:28) ~[commons-core-2.0.jar:?]
        at com.mixky.asp.commons.design.cache.impl.DesignResourceCacheImpl.listAllObjects(DesignResourceCacheImpl.java:161) ~[commons-core-2.0.jar:?]
        at com.mixky.asp.commons.design.cache.impl.DesignResourceCacheImpl.listAllObjects(DesignResourceCacheImpl.java:65) ~[commons-core-2.0.jar:?]
        at com.mixky.asp.commons.design.DesignObjectFactory.listAllObjects(DesignObjectFactory.java:287) ~[commons-core-2.0.jar:?]
        at com.mixky.asp.framework.service.impl.ApplicationDesignObjectServiceImpl.listGridDesignObject(ApplicationDesignObjectServiceImpl.java:332) [framework-2.0.jar:?]
        at com.mixky.asp.framework.service.ApplicationService.listGridDesignObject(ApplicationService.java:300) [framework-2.0.jar:?]
        at com.mixky.asp.framework.rpc.service.designobject.impl.DesignObjectRPCServiceImpl.listGridDesignObject(DesignObjectRPCServiceImpl.java:132) [framework-2.0.jar:?]
        at com.mixky.asp.framework.rpc.service.designobject.DesignObjectRPCService$Processor$listGridDesignObject.getResult(DesignObjectRPCService.java:2185) [framework-2.0.jar:?]
        at com.mixky.asp.framework.rpc.service.designobject.DesignObjectRPCService$Processor$listGridDesignObject.getResult(DesignObjectRPCService.java:2170) [framework-2.0.jar:?]
        at org.apache.thrift.ProcessFunction.process(ProcessFunction.java:39) [libthrift-0.9.3.jar:0.9.3]
        at org.apache.thrift.TBaseProcessor.process(TBaseProcessor.java:39) [libthrift-0.9.3.jar:0.9.3]
        at org.apache.thrift.TMultiplexedProcessor.process(TMultiplexedProcessor.java:123) [libthrift-0.9.3.jar:0.9.3]
        at org.apache.thrift.server.AbstractNonblockingServer$FrameBuffer.invoke(AbstractNonblockingServer.java:518) [libthrift-0.9.3.jar:0.9.3]
        at org.apache.thrift.server.Invocation.run(Invocation.java:18) [libthrift-0.9.3.jar:0.9.3]
        at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142) [?:1.8.0_131]
        at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617) [?:1.8.0_131]
        at java.lang.Thread.run(Thread.java:748) [?:1.8.0_131]

```

##### 定位代码

```java
	public static <T extends AbstractDesignObject> void sortListWithOrderField(List<T> values) {
		Collections.sort(values,new Comparator<T>() {

			@Override
			public int compare(T o1, T o2) {
				//进行排序
				Integer order1 = o1.getOrder(); //读取第一个对象的order值
				Integer order2 = o2.getOrder(); //读取第二个对象的order值
				if(order1 != null && order2 != null) {
					return order1.compareTo(order2);
				}
				return 0;
			}

		});
	}
```

##### 解决问题

###### 第一种方式 - 不修改排序代码

设置系统属性：
``` java
System.setProperty("java.util.Arrays.useLegacyMergeSort", "true");
```

###### 第二种方式 - 修改排序代码

```
public static <T extends AbstractDesignObject> void sortListWithOrderField(List<T> values) {
		Collections.sort(values,new Comparator<T>() {

			@Override
			public int compare(T o1, T o2) {
				//进行排序
				Integer order1 = o1.getOrder(); //读取第一个对象的order值
				Integer order2 = o2.getOrder(); //读取第二个对象的order值
				
				// 如果都不为空，按order升序排序，否则空值排列在后面
				if(order1 != null && order2 != null) {
					return order1.compareTo(order2);
				} else if (order1 != null) {
					return -1;
				} else if (order2 != null) {
					return 1;
				} else {
					return 0;
				}
			}

		});
```

##### 问题分析

从JDK7开始，默认排序使用TimSort,而以前的版本使用的是LegacyMergeSort，TimSort的排序要符合一些规则，官方说明如下：

>Compares its two arguments for order. Returns a negative integer, zero, or a positive integer as the first argument is less than, equal to, or greater than the second.In the foregoing description, the notation sgn(expression) designates the mathematical signum function, which is defined to return one of -1, 0, or 1 according to whether the value ofexpression is negative, zero or positive. The implementor must ensure that sgn(compare(x, y)) == -sgn(compare(y, x)) for all x and y. (This implies that compare(x, y) must throw an exception if and only if compare(y, x) throws an exception.) The implementor must also ensure that the relation is transitive: ((compare(x, y)>0) && (compare(y, z)>0)) implies compare(x, z)>0. Finally, the implementor must ensure that compare(x, y)==0 implies that sgn(compare(x, z))==sgn(compare(y, z)) for all z. It is generally the case, but not strictly required that (compare(x, y)==0) == (x.equals(y)). Generally speaking, any comparator that violates this condition should clearly indicate this fact. The recommended language is “Note: this comparator imposes orderings that are inconsistent with equals.”

意思就是排序代码需要符合三个规则：

>对任意对象x,y,z必须满足：  
1、sgn(compare(x, y)) == -sgn(compare(y, x))  
2、如果compare(x, y)>0，并且compare(y, z)>0， 意味着 compare(x, z)>0   
3、如果compare(x, y)==0 意味着 sgn(compare(x, z))==sgn(compare(y, z))


所以最开始的代码违反了规则3：  
compare(null,2) = 0  但是 sgn(compare(null, 3)) = 0 != sgn(compare(2, 3)) = -1

修改后的代码就符合上面三个规则了。

而第一种解决方式是改用以前JDK的实现方式。
