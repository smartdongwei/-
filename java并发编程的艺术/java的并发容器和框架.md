# java的并发容器和框架

## 1 ConcurrentHashMap的实现原理和使用

### 1.1 使用ConcurrentHashMap的原因

​    在并发编程中使用HashMap可能导致程序死循环(多线程会导致HashMap的Entry链表形成环形数据结构，一旦形成，则Entry的next节点永远不为空，就会产生死循环获取entry)，使用线程安全的HashTable效率低下(使用synchronized来保证线程安全)。

​    而ConcurrentHashMap使用的是锁分段技术，将数据分成一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一段数据时，其它段数据也能被使用。

### 1.2 ConcurrentHashMap相关原理

​    ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁，在ConcurrentHashMap里扮演锁的角色，HashEntry则用于存储键值对数据。每一个Segment守护着一个HashEntry数组里的元素，每次对其进行修改时，都需要先获取与它对应的Segment锁。

#### 1.2.1 ConcurrentHashMap的初始化







## 2 Fork/Join框架

   是一个把大任务分割成若干小任务，最终汇总每一个小任务结果后得到大任务结果的框架。

### 2.1 工作窃取算法

​    该算法是指某个线程从其它队列里窃取任务来执行，当一个线程完成分配的队列里面的任务时，可以去其它线程的队列中窃取一个任务来执行。而这时他们会访问同一个队列，所以为了减少不同线程之间的竞争，通常会使用双端队列，被窃取任务线程永从双端队列的头部拿任务执行，而窃取任务的线程永远从双端队列的尾部拿任务执行。

### 2.2 Fork/Join 框架的设计

​    **步骤1 分割任务**。首先需要一个fork类来把大任务分割成小任务。

​    **步骤2 执行任务并合并结果。**分割的子任务放在双端队列中，然后几个启动线程分别从双端队列中获取任务执行。子任务执行完的结果分别都放在一个队列中，启动一个线程从队列中拿数据，然后再合并这些数据。

  1: ForkJoinTask: 要使用这个框架，必须创建一个ForkJoin任务。Fork/Join使用以下两个子类来完成相关任务：

- RecursiveAction: 用于没有返回结果的任务。
- RecursiveTask: 用于有返回结果的任务。

  2：ForkJoinPool: ForkJoinTask需要通过ForkJoinPool来执行。

### 2.3 使用Fork/Join框架

#### 2.3.1：简单的累加程序

```java
package com.company.wdwstudy;

import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;

public class CountTask extends RecursiveTask<Integer> {
    // 阈值
    private static final int THRESHOLD = 20;
    private int start;
    private int end;
    public CountTask(int start,int end){
        this.start = start;
        this.end = end;
    }
    @Override
    protected Integer compute() {
        int sum = 0;
        boolean canCompute = (end - start) < THRESHOLD;
        if(canCompute){
            for( int i = start; i<= end; i++){
                sum += i;
            }
        }else{
            int middle = (start + end ) /2;
            CountTask leftTask = new CountTask(start, middle);
            CountTask rightTask = new CountTask(middle +1, end);
            // 执行子任务
            leftTask.fork();
            rightTask.fork();
            int leftResult = leftTask.join();
            int rightResult = rightTask.join();
            // 合并子任务
            sum = leftResult + rightResult;

        }
        return sum;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinPool forkJoinPool = new ForkJoinPool();
        CountTask task = new CountTask(1,1000);
        Future<Integer> result = forkJoinPool.submit(task);
        System.out.println(result.get());
    }
}
```



#### 2.3.2 归并排序

```java
package com.company.wdwstudy;

import java.util.Arrays;
import java.util.Random;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;

public class Merge2 {
    private static int MAX = 100000000;

    private static int inits[] = new int[MAX];

    // 同样进行随机队列初始化，这里就不再赘述了
    static {
        Random r = new Random();
        for(int index = 1 ; index <= MAX ; index++) {
            inits[index - 1] = r.nextInt(10000000);
        }

    }

    public static void main(String[] args) throws Exception {
        // 正式开始
        long beginTime = System.currentTimeMillis();
        ForkJoinPool pool = new ForkJoinPool();
        MyTask task = new MyTask(inits);
        ForkJoinTask<int[]> taskResult = pool.submit(task);
        try {
            taskResult.get();
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace(System.out);
        }
        long endTime = System.currentTimeMillis();
        System.out.println("耗时=" + (endTime - beginTime));
    }

    /**
     * 单个排序的子任务
     * @author yinwenjie
     */
    static class MyTask extends RecursiveTask<int[]> {

        private int source[];

        public MyTask(int source[]) {
            this.source = source;
        }

        /* (non-Javadoc)
         * @see java.util.concurrent.RecursiveTask#compute()
         */
        @Override
        protected int[] compute() {
            int sourceLen = source.length;
            // 如果条件成立，说明任务中要进行排序的集合还不够小
            if(sourceLen > 2) {
                int midIndex = sourceLen / 2;
                // 拆分成两个子任务
                MyTask task1 = new MyTask(Arrays.copyOf(source, midIndex));
                //task1.fork();
                MyTask task2 = new MyTask(Arrays.copyOfRange(source, midIndex , sourceLen));
                // task2.fork();
                invokeAll(task1,task2);
                // 将两个有序的数组，合并成一个有序的数组
                int result1[] = task1.join();
                int result2[] = task2.join();
                int mer[] = joinInts(result1 , result2);
                return mer;
            }
            // 否则说明集合中只有一个或者两个元素，可以进行这两个元素的比较排序了
            else {
                // 如果条件成立，说明数组中只有一个元素，或者是数组中的元素都已经排列好位置了
                if(sourceLen == 1
                        || source[0] <= source[1]) {
                    return source;
                } else {
                    int targetp[] = new int[sourceLen];
                    targetp[0] = source[1];
                    targetp[1] = source[0];
                    return targetp;
                }
            }
        }

        private int[] joinInts(int array1[] , int array2[]) {
            // 和上文中出现的代码一致
            int destInts[] = new int[array1.length + array2.length];
            int array1Len = array1.length;
            int array2Len = array2.length;
            int destLen = destInts.length;

            // 只需要以新的集合destInts的长度为标准，遍历一次即可
            for(int index = 0 , array1Index = 0 , array2Index = 0 ; index < destLen ; index++) {
                int value1 = array1Index >= array1Len?Integer.MAX_VALUE:array1[array1Index];
                int value2 = array2Index >= array2Len?Integer.MAX_VALUE:array2[array2Index];
                // 如果条件成立，说明应该取数组array1中的值
                if(value1 < value2) {
                    array1Index++;
                    destInts[index] = value1;
                }
                // 否则取数组array2中的值
                else {
                    array2Index++;
                    destInts[index] = value2;
                }
            }

            return destInts;
        }
    }

}
```