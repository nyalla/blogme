---
template: blog-post
title: Java 8 parallel processing with completable future
slug: Java-8-parallel-processing-with-completable-future
date: 2020-07-09 15:55
description: |-
  
  Java 8 parallel processing with completable future
---
Everyday we see the programming use cases which should handle data-parallel to improve the performance. From Java 8, with CompletableFuture we can achieve parallel programming much simpler and readable way with methods like allOf , join etc..

Whenever we calling any method we have to decide whether we need any return value from that method or not. If we don't want any return value we can just call that method and leave the control to it. Because in CompletableFuture we have 2 methods which addresses the above cases.

1. CompletableFuture.supplyAsync -- In case if you want the return value
2. CompletableFuture.runAsync -- In case if you don't want the return value

So let's take an example, we are taking 3 tasks that have to be executed parallel.

* Method 1: add -> it takes the arguments of 2 variable and returns the sum

  ```
  public static  Integer  addFun1(int a, int b) {          
  System.out.println(Thread.currentThread().getName());
    for (int i=0;i<10;i++){
      System.out.print(Thread.currentThread().getName()+i);
             }
         return  a+b ;
         }
  ```
* Method 2: sub-> it takes the arguments of 2 variable and returns the subtraction

  ```
  public static  Integer  subFun1(int a, int b) {
  System.out.println(Thread.currentThread().getName());
      for (int i=0;i<10;i++){
          System.out.print(Thread.currentThread().getName()+i);
              }
         return  a-b ;
      }
  ```
* Method 3: mul-> it takes the arguments of 2 variable and returns the multiplication

  ```
  public static  Integer  mulFun1(int a, int b) {
  	System.out.println(Thread.currentThread().getName());
              for (int i=0;i<10;i++){
                  System.out.print(Thread.currentThread().getName()+i);
              }
              return  a*b ;
          }
  ```

Actual CompletableFuture work starts from here. Before adding futures we have to maintain a global list which bundles all the future list.

`List<CompletableFuture<Integer>> futuresList = new ArrayList<CompletableFuture<Integer>>();`

Then the method which we have created that has to be run parallel we have declare them as CompletableFuture methods.

```
CompletableFuture<Integer> addAsy = CompletableFuture.supplyAsync(()->(addFun1(10,5)));
CompletableFuture<Integer> subAsy = CompletableFuture.supplyAsync(()->(subFun1(10,5)));
CompletableFuture<Integer> mulAsy = CompletableFuture.supplyAsync(()->(mulFun1(10,5)));
```

So the above methods are converted into completable futures that can process parallel.

As we said in the above line all the futures are added to the global future list.

```
futuresList.add(addAsy);
futuresList.add(subAsy);
futuresList.add(mulAsy);
```

Then we have to write a line that keeps the agreement among all threads saying "waits till all the threads in the arguments get completed". For that, we have to use "allOf" method.

```
CompletableFuture<Void> allFutures = CompletableFuture
                .allOf(futuresList.toArray(new CompletableFuture[futuresList.size()]));
```

Then we have to write a line that says after the completion of execution for all threads, collect all the return values from all the threads.

```
CompletableFuture<List<Integer>> allCompletableFuture = allFutures.thenApply(future -> {
            return futuresList.stream().map(completableFuture -> completableFuture.join())
                    .collect(Collectors.toList());
        });
```

Finally, define the future list as a completable future.

```
CompletableFuture<List<Integer>> completableFuture = allCompletableFuture.toCompletableFuture();
```

All the setup has done for making the parallel processing using completable future. To call that entire parallel method, below is the line using get()

```
try {
            List<Integer> finalList = (List<Integer>) completableFuture.get();
            System.out.print(finalList);
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
```

You will get all the results in finalList variable. To get the things more clear, I have written for loop inside each method to process sysout multiple times to make each thread more time in that method.
After executing the program just analyze the sysout comments and trace the control for better understanding.

Code is available in Github [link](https://raw.githubusercontent.com/nyalla/attachments/master/ParallelProcessing.java).