title: 集成spring和quartz
date: 2013-09-24 11:05
categories: java 
---
quartz是个好东西，今天用的版本是quartz-1.7.3
<!--more-->

首先需要写一个工作类，继承自QuartzJobBean，这个类的作用类似于TimerTask

```
public class MyTimer extends QuartzJobBean {

	@Override
	protected void executeInternal(JobExecutionContext context)
			throws JobExecutionException {

		System.out.println("gogo");

	}
}
```

这里的executeInternal()就是会定时执行的方法。然后是spring的配置文件：

```
<?xml version="1.0" encoding="UTF-8"?>

<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans 
	http://www.springframework.org/schema/beans/spring-beans.xsd">

	<bean id="myTimerJob" class="org.springframework.scheduling.quartz.JobDetailBean">
		<property name="jobClass" value="xxx.xxx.xxx.MyTimer" />
	</bean>

	<bean id="myTimerTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
		<property name="jobDetail" ref="myTimerJob" />
		<property name="cronExpression" value="1 * * * * ?" />
	</bean>

	<bean id="schedulerFactory"
		class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
		<property name="triggers">
			<list>
				<ref bean="myTimerTrigger" />
			</list>
		</property>
	</bean>

</beans>
```

上面分别定义了3个bean，是嵌套的关系 

首先schedulerFactory这个bean，类型是org.springframework.scheduling.quartz.SchedulerFactoryBean，是作为一个总的调度，放在triggers列表中的触发器，就会被触发 

然后myTimerTrigger这个bean，类型是org.springframework.scheduling.quartz.CronTriggerBean，是为每个定时任务设置触发时间，这样的bean会有多个，与定时任务是一一对应的 

最后myTimerJob这个bean，类型是org.springframework.scheduling.quartz.JobDetailBean，也就是实际的TimerTask，这样的bean也会有多个，与Trigger是一一对应的