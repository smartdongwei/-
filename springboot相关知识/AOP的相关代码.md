# AOP的相关代码

```java
@Around("hazecastLockMethod()")
public Object hazecastLockMethod(ProceedingJoinPoint pjp) throws Throwable {
    Object result = null;
    Signature signature = pjp.getSignature();
    try{
        //获取接口的参数信息
        MethodSignature methodSignature = (MethodSignature) signature;
        Method method = methodSignature.getMethod();
        if(method == null){
            return null;
        }
        HazecastLock hazecastLock = method.getAnnotation(HazecastLock.class);
        if(hazecastLock == null){
            return null;
        }
        String lockName = hazecastLock.lockName();
        String methodValue = hazecastLock.methodValue();
        if(StringUtils.isBlank(lockName)){
            result = pjp.proceed();
            return result;
        }
        Lock lock = hazelcastInstance.getCPSubsystem().getLock(lockName);
        if(lock.tryLock()){
            try {
                logger.info("定时任务【"+methodValue+"】开始运行----------------");
                result = pjp.proceed();
                logger.info("定时任务【"+methodValue+"】运行结束--------------");
                return result;
            }catch (Exception e){
                logger.error(ExceptionUtil.getExceptionTrace(e));
                throw new Exception(ExceptionUtil.getExceptionTrace(e));
            }finally {
                lock.unlock();
            }
        }else{
            logger.info("定时任务【"+methodValue+"】争抢锁失败，定时任务不是由该机器运行");
            return null;
        }
    }catch (Exception e){
        logger.error(ExceptionUtil.getExceptionTrace(e));
        return null;
    }

}
```