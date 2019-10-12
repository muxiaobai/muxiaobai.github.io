---
title: java-常用代码2-future等
date: 2019-09-29 20:33:32
tags:
categories: java
description: "java常用代码汇总，读futuretask,callable,使用"
---


远程调用可以合并 10ms执行一次。阻塞队列

```

@Service
public class DemoMoreService {

    @Autowired
    private RemoteServiceCall remoteCall;
    //阻塞队列
    LinkedBlockingQueue<Request> queue = new LinkedBlockingQueue<Request>();
    
    public Map<String, Object> doRemote(String orderCode) throws InterruptedException, ExecutionException{
        
        Request request = new Request();
        request.orderCode = orderCode;
        //jdk1.8
        CompletableFuture<Map<String, Object>> future = new CompletableFuture<>();
        request.future = future;
        //队列中
        queue.add(request);
        return future.get();//阻塞状态
    }
    /**
     * 定时任务 初始化执行
     * init:(). servlet init方法之前调用这个注解的方法，只会被调用一次
     * @author Mu Xiaobai
     * @since JDK 1.8
     */
    @PostConstruct
    public void init(){
        //定时任务两个线程数
        ScheduledExecutorService scheduledExecutorService =Executors.newScheduledThreadPool(2);
       
        scheduledExecutorService.scheduleAtFixedRate(()->{
           
            //run
            int size = queue.size();
            if(size == 0){
                return;
            }
            //弹出Request
            List<Request> requests = new ArrayList<>();
            for(int i=0; i<size;i++){
                Request request = queue.poll();//出队列
                requests.add(request);
            }
          
            System.out.println("10ms 取到的本地请求数："+size);
            
            //循环requests分离orderCode和future
            List<String> orderCodes = new ArrayList<>();
            for(Request request :requests){
                orderCodes.add(request.orderCode);
            }
            
            //查询返回结果
            List<Map<String,Object>> responses=remoteCall.getMore(orderCodes);
            System.out.println(responses);
            //分离返回内容
            Map<String,Map<String,Object>>  responseMap = new HashMap<>(); 
            for(Map<String,Object> response:responses){
                String orderCode = response.get("orderCode").toString();
                responseMap.put(orderCode, response);
            }
            //结果转发到request
            for(Request request:requests){
                Map<String, Object> result = responseMap.get(request.orderCode);
                request.future.complete(result);//转发到对应的线程
            }
            
        }, 0, 10, TimeUnit.MILLISECONDS); //10ms执行
        
    }
    class Request{
        String orderCode;
        CompletableFuture<Map<String, Object>> future;
    }
}
```
远程接口
```
/**
 * Project Name:spring-boot
 * File Name:remoteServiceCall.java
 * Package Name:io.github.muxiaobai.spring_boot.remoteService
 * Date:2019年3月22日上午10:37:05
 * Copyright (c) 2019, All Rights Reserved.
 *
*/

package io.github.muxiaobai.spring_boot.remoteService;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import org.springframework.stereotype.Service;

/**
 * ClassName:remoteServiceCall 
 * Function: TODO 
 * Reason:	 TODO 
 * Date:     2019年3月22日 上午10:37:05 
 * @author   Mu Xiaobai
 * @version  
 * @since    JDK 1.8	 
 */
@Service
public class RemoteServiceCall {
    public Map<String,Object> getOne(String orderCode){
        Map<String,Object> map = new HashMap<>();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            
            // TODO Auto-generated catch block
            e.printStackTrace();
            
        }
        map.put("orderCode", orderCode);
        map.put("hello", "hello");
        return map ;
        
    }
    public Map<String,Object> getTwo(String orderCode){
        Map<String,Object> map = new HashMap<>();
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            
            // TODO Auto-generated catch block
            e.printStackTrace();
            
        }
        map.put("orderCode2", orderCode);
        map.put("hello2", "hello2");
        return map ;
        
    }
    public List<Map<String,Object>> getMore(List<String> orderCodes){
        List<Map<String, Object>> list = new ArrayList<Map<String,Object>>();
        for(String orderCode: orderCodes){
            Map<String,Object> map = new HashMap<>();
            map.put("code", orderCode);
            map.put("hello", "hello");
            list.add(map);
        }
        return list ;
        
    }
}


```
#### 多线程  串行，并行futuretask  线程池并行

```
/**
 * Project Name:spring-boot
 * File Name:DemoService.java
 * Package Name:io.github.muxiaobai.spring_boot.service
 * Date:2019年3月21日下午7:31:58
 * Copyright (c) 2019, All Rights Reserved.
 *
*/

package io.github.muxiaobai.spring_boot.service;

import io.github.muxiaobai.spring_boot.remoteService.RemoteServiceCall;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.*;

/**
 * ClassName:DemoService 
 * Function: TODO 
 * Reason:	 TODO 
 * Date:     2019年3月21日 下午7:31:58 
 * @author   Mu Xiaobai
 * @version  
 * @since    JDK 1.8	 
 */

@Service
public class DemoMoreThreadService {

    @Autowired
    private RemoteServiceCall remoteCall;
    
    protected static ExecutorService threads = Executors.newFixedThreadPool(10);
    /**
     * 使用线程池并行
     * doExecPoolRemote:().
     * @author Mu Xiaobai
     * @param orderCode
     * @return
     * @throws InterruptedException
     * @throws ExecutionException
     * @since JDK 1.8
     */
    @Transactional(propagation = Propagation.REQUIRED)
    public Map<String, Object> doExecPoolRemote(String orderCode) throws InterruptedException, ExecutionException{
        System.out.println("sssss");
        Callable<Map<String, Object>> callable =  new Callable<Map<String, Object>>() {
            @Override
            public Map<String, Object> call() throws Exception {
                return remoteCall.getOne(orderCode);
            }
        }; 
      
        Callable<Map<String, Object>> callable1 =  new Callable<Map<String, Object>>() {
            @Override
            public Map<String, Object> call() throws Exception {
                return remoteCall.getTwo(orderCode);
            }
        }; 
        FutureTask<Map<String, Object>> futureTask = new FutureTask<>(callable);
        FutureTask<Map<String, Object>> futureTask1 = new FutureTask<>(callable1);
        
        threads.submit(futureTask);
        threads.submit(futureTask1);
        
        Map<String, Object> result = new HashMap<>();
        result.putAll(futureTask.get());
        result.putAll(futureTask1.get());
        return result;
}
    /**
     * 使用线程并行
     * doThreadRemote:().
     * @author Mu Xiaobai
     * @param orderCode
     * @return
     * @throws InterruptedException
     * @throws ExecutionException
     * @since JDK 1.8
     */
    public Map<String, Object> doThreadRemote(String orderCode) throws InterruptedException, ExecutionException{
        
        Callable<Map<String, Object>> callable =  new Callable<Map<String, Object>>() {
            @Override
            public Map<String, Object> call() throws Exception {
                return remoteCall.getOne(orderCode);
            }
        }; 
        
        Callable<Map<String, Object>> callable1 =  new Callable<Map<String, Object>>() {
            @Override
            public Map<String, Object> call() throws Exception {
                return remoteCall.getTwo(orderCode);
            }
        }; 
        
        FutureTask<Map<String, Object>> futureTask = new FutureTask<>(callable);
        new Thread(futureTask).start();
        FutureTask<Map<String, Object>> futureTask1 = new FutureTask<>(callable1);
        new Thread(futureTask1).start();
        
        Map<String, Object> result = new HashMap<>();
        result.putAll(futureTask.get());
        result.putAll(futureTask1.get());
        return result;
    }
    /**
     * 串行
     * doEachRemote:().
     * @author Mu Xiaobai
     * @param orderCode
     * @return
     * @throws InterruptedException
     * @throws ExecutionException
     * @since JDK 1.8
     */
    public Map<String, Object> doEachRemote(String orderCode) throws InterruptedException, ExecutionException{
       
        Map<String, Object> result = new HashMap<>();
        result.putAll(remoteCall.getOne(orderCode));
        result.putAll(remoteCall.getTwo(orderCode));
       
        return result;
    }
 
}


```