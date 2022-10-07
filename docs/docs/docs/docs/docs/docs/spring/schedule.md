1. [spring schedule定时任务详解](https://blog.csdn.net/qq_34480904/article/details/122410711)
2. [@Scheduled(cron = "* * * * * *") cron表达式详解](https://blog.csdn.net/m0_37179470/article/details/81271607)

## 1 spring的schedule定时任务

### 1.1 使用方法

**1 启动类使用@EnableScheduling注解开启定时任务**

```
@SpringBootApplication
@EnableScheduling
public class ScheduledTest {
    public static void main(String[] args) {
        SpringApplication.run(ScheduledTest.class);
    }
}
```

**2 添加定时任务**

* 2.1 使用@Scheduled注解

```
@Scheduled(cron = "0/2 * * * * ? ")
public void index1() {
    log.info("定时任务1，2秒执行一次，time：" + DateTime.now() + " 线程：" + Thread.currentThread().getName());
}
```

* 2.2 实现SchedulingConfigurer接口

```
@Configuration
@Component
@Slf4j
public class TestTask implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        taskRegistrar.addFixedDelayTask(this::index2, 1000);
    }
    public void index2() {
        log.info("定时任务2，1秒执行一次，time：" + DateTime.now() + " 线程：" + Thread.currentThread().getName());
    }
}
```

### 1.2 cron表达式

#### 1.2.1 格式

`{秒数} {分钟} {小时} {日期} {月份} {星期} {年份(可为空)}`

#### 1.2.2 占位符解释

占位符|解释
:-|:-
{秒数}{分钟} | 允许值范围: 0~59 ,不允许为空值，若值不合法，调度器将抛出SchedulerException异常
`*` | 代表每隔1秒钟触发；
`,` | 代表在指定的秒数触发，比如”0,15,45”代表0秒、15秒和45秒时触发任务
`-` | 代表在指定的范围内触发，比如”25-45”代表从25秒开始触发到45秒结束触发，每隔1秒触发1次
`/` | 代表触发步进(step)，”/”前面的值代表初始值(““等同”0”)，后面的值代表偏移量，比如”0/20”或者”/20”代表从0秒钟开始，每隔20秒钟触发1次，即0秒触发1次，20秒触发1次，40秒触发1次；”5/20”代表5秒触发1次，25秒触发1次，45秒触发1次；”10-45/20”代表在[10,45]内步进20秒命中的时间点触发，即10秒触发1次，30秒触发1次
`{小时}` | 允许值范围: 0~23 ,不允许为空值，若值不合法，调度器将抛出SchedulerException异常,占位符和秒数一样
`{日期}` | 允许值范围: 1~31 ,不允许为空值，若值不合法，调度器将抛出SchedulerException异常
`{星期}` | 允许值范围: 1~7 (SUN-SAT),1代表星期天(一星期的第一天)，以此类推，7代表星期六(一星期的最后一天)，不允许为空值，若值不合法，调度器将抛出SchedulerException异常
`{年份}` | 允许值范围: 1970~2099 ,允许为空，若值不合法，调度器将抛出SchedulerException异常

注意：除了`{日期}`和`{星期}`可以使用`?`来实现互斥，表达无意义的信息之外，其他占位符都要具有具体的时间含义，且依赖关系为：`年->月->日期(星期)->小时->分钟->秒数`

#### 1.2.3 案例

例子|解释
:-|:-
`30 * * * * ?` |每半分钟触发任务
`30 10 * * * ?` |每小时的10分30秒触发任务
`30 10 1 * * ?` |每天1点10分30秒触发任务
`30 10 1 20 * ?` |每月20号1点10分30秒触发任务
`30 10 1 20 10 ? *` |每年10月20号1点10分30秒触发任务
`30 10 1 20 10 ? 2011` |2011年10月20号1点10分30秒触发任务
`30 10 1 ? 10 * 2011` |2011年10月每天1点10分30秒触发任务
`30 10 1 ? 10 SUN 2011` |2011年10月每周日1点10分30秒触发任务
`15,30,45 * * * * ?` |每15秒，30秒，45秒时触发任务
`15-45 * * * * ?` |15到45秒内，每秒都触发任务
`15/5 * * * * ?` |每分钟的每15秒开始触发，每隔5秒触发一次
`15-30/5 * * * * ?` |每分钟的15秒到30秒之间开始触发，每隔5秒触发一次
`0 0/3 * * * ?` |每小时的第0分0秒开始，每三分钟触发一次
`0 15 10 ? * MON-FRI` |星期一到星期五的10点15分0秒触发任务
`0 15 10 L * ?` |每个月最后一天的10点15分0秒触发任务
`0 15 10 LW * ?` |每个月最后一个工作日的10点15分0秒触发任务
`0 15 10 ? * 5L` |每个月最后一个星期四的10点15分0秒触发任务
`0 15 10 ? * 5#3` |每个月第三周的星期四的10点15分0秒触发任务

### 1.3 使定时任务多线程非阻塞运行

#### 1.3.1 默认情况下阻塞的原因

默认情况，定时任务使用的是单例线程执行器Executors.newSingleThreadScheduledExecutor()，所以当一个定时任务阻塞是，所有定时任务都不会执行。

#### 1.3.2 解决

**实现SchedulingConfigurer接口，设置任务调度器实现类。**

```
@Configuration
@Component
@Slf4j
public class TestTask implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar taskRegistrar) {
        ThreadFactory nameThreadFactory = new ThreadFactoryBuilder().setNameFormat("scheduled-%d").build();
        taskRegistrar.setScheduler(new ScheduledThreadPoolExecutor(5,
                nameThreadFactory,
                new ThreadPoolExecutor.AbortPolicy())); //丢弃任务并抛出RejectedExecutionException异常
    }
}
```

### 1.4 spring schedule原理--整个流程解析

#### 1.4.1 从启动类上的@EnableScheduling开始

```
//进入EnableScheduling注解
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Import(SchedulingConfiguration.class)  //引用SchedulingConfiguration.class
@Documented
public @interface EnableScheduling {}

//进入SchedulingConfiguration类
@Configuration(proxyBeanMethods = false)
@Role(BeanDefinition.ROLE_INFRASTRUCTURE)
public class SchedulingConfiguration {

   @Bean(name = TaskManagementConfigUtils.SCHEDULED_ANNOTATION_PROCESSOR_BEAN_NAME)
   @Role(BeanDefinition.ROLE_INFRASTRUCTURE)
   public ScheduledAnnotationBeanPostProcessor scheduledAnnotationProcessor() {
      return new ScheduledAnnotationBeanPostProcessor();    //实例化ScheduledAnnotationBeanPostProcessor类
   }
}

//进入ScheduledAnnotationBeanPostProcessor类，
//它实现了很多接口，后面只看BeanPostProcessor和ApplicationListener
//只看内部的构造方法，registrar会用到
public ScheduledAnnotationBeanPostProcessor() {
    this.registrar = new ScheduledTaskRegistrar();
}
```

#### 1.4.2 BeanPostProcessor接口

调用postProcessAfterInitialization后置处理器：

1. 扫描注解（扫描@Scheduled、@Schedules注解），
2. 全部转换为Scheduled后，
3. 调用`processScheduled`方法

processScheduled方法内部：

1. 先创建执行runnable
2. 获取延迟执行时间
3. 获取cron表达式，创建CronTask，registrar中添加任务
4. 获取固定延迟，创建FixedDelayTask，registrar中添加任务
5. 获取固定执行间隔，创建FixedRateTask，registrar中添加任务
6. 把所有任务都添加到`scheduledTasks` **【重点】**

#### 1.4.3 ApplicationListener接口

工程启动好后调用onApplicationEvent方法：

1. 扫描所有的SchedulingConfigurer实现类，（如上面自定义的实现类）
2. 调用configureTasks回调函数添加定时任务。（如上面自定义的实现类中的实现方法）
3. 调用 registrar 的 `afterPropertiesSet` 方法。

afterPropertiesSet方法内部在**给线程池中安排任务**了：

```
@Override
public void afterPropertiesSet() {
   scheduleTasks();
}

/**
 * Schedule all registered tasks against the underlying
 * {@linkplain #setTaskScheduler(TaskScheduler) task scheduler}.
 */
@SuppressWarnings("deprecation")
protected void scheduleTasks() {
   if (this.taskScheduler == null) {
      this.localExecutor = Executors.newSingleThreadScheduledExecutor();
      this.taskScheduler = new ConcurrentTaskScheduler(this.localExecutor);
   }
   if (this.triggerTasks != null) {
      for (TriggerTask task : this.triggerTasks) {
         addScheduledTask(scheduleTriggerTask(task));   //下面以这个为例子
      }
   }
   if (this.cronTasks != null) {
      for (CronTask task : this.cronTasks) {
         addScheduledTask(scheduleCronTask(task));
      }
   }
   if (this.fixedRateTasks != null) {
      for (IntervalTask task : this.fixedRateTasks) {
         addScheduledTask(scheduleFixedRateTask(task));
      }
   }
   if (this.fixedDelayTasks != null) {
      for (IntervalTask task : this.fixedDelayTasks) {
         addScheduledTask(scheduleFixedDelayTask(task));
      }
   }
}

//以scheduleTriggerTask为例，将任务放进线程池
@Nullable
public ScheduledTask scheduleTriggerTask(TriggerTask task) {
   ScheduledTask scheduledTask = this.unresolvedTasks.remove(task);
   boolean newTask = false;
   if (scheduledTask == null) {
      scheduledTask = new ScheduledTask(task);
      newTask = true;
   }
   if (this.taskScheduler != null) {
      scheduledTask.future = this.taskScheduler.schedule(task.getRunnable(), task.getTrigger());
   }
   else {
      addTriggerTask(task);
      this.unresolvedTasks.put(task, scheduledTask);
   }
   return (newTask ? scheduledTask : null);
}
private void addScheduledTask(@Nullable ScheduledTask task) {
    if (task != null) {
        this.scheduledTasks.add(task);
    }
}
```