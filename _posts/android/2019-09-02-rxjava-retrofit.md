---
layout:     post
title:      "响应式编程 rxjava"
subtitle:   "rxjava & retrofit"
date:       2019-09-02 16:17:15
author:     "zhouhaoo"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
---

响应式函数编程_RxJava & RxAndroid，最近越来火的rx系列，响应式的编程 在程序逻辑异常复杂的情况下,仍然可以让代码的逻辑保持简洁。
<!--more-->
相关介绍：
* 主页: [https://github.com/ReactiveX/RxJava](https://github.com/ReactiveX/RxJava)
* 中文资料: 
	* [https://github.com/lzyzsd/Awesome-RxJava](https://github.com/lzyzsd/Awesome-RxJava)
	* [https://www.zhihu.com/question/35511144](https://www.zhihu.com/question/35511144)
* 用途:
	* 异步操作
	* 在程序逻辑异常复杂的情况下,仍然可以让代码的逻辑保持简洁
* 配置: 添加依赖: 
	* compile 'io.reactivex:rxjava:1.1.3'
	* compile 'io.reactivex:rxandroid:1.1.0'
	* 如果结合Retrofit使用,需要添加以下依赖
	* compile 'com.squareup.retrofit2:retrofit:2.0.1'
	* compile 'com.squareup.retrofit2:converter-gson:2.0.1'
	* compile 'com.squareup.retrofit2:adapter-rxjava:2.0.1'

* 基本概念:
	1. 被观察者: Observable  
		* 作用: 决定什么时候触发事件以及触发怎样的事件
		* 创建方法:
			* Observable.just(T...) 参数为单个的
			* Observable.from(T[]) / Observable.from(Iterable<? extends T>)  参数为数组或Iterable
	2. 观察者: Observer 
		* 作用: 当事件触发的时候将有怎样的行为
		* 实现类有Observer / Subscriber 
	3. 订阅: subscribe 
		* 作用: 把Observable和Observer关联起来
		* 方法:
			* observable.subscribe(observer);
			* observable.subscribe(subscriber);
	4. 事件:
		* onNext()：普通事件
		* onCompleted():事件队列完结
		* onError(): 事件队列异常
		* 需要注意的是onCompleted()和onError()是互斥的.调用了其中一个就不应该触发另一个.
	5. 案例:
		* 现有一个数组 String[] arr ={"afdsa", "bfdsa", "cfda"}, 把其中以字母"a"开头的字符串找出来并加上"from Alpha",最后打印出新的字符串的长度
		
			    private void simpleDemo() {
			
			        String[] arr = {"afdsa", "bfdsa", "cfda"};
			
			        Observable
			                .from(arr)
			                .filter(new Func1<String, Boolean>() {
			                    @Override
			                    public Boolean call(String s) {
			                        return s.startsWith("a");
			                    }
			                })
			                .map(new Func1<String, String>() {
			                    @Override
			                    public String call(String s) {
			                        return s + " from Alpha";
			                    }
			                })
			                .subscribe(new Action1<String>() {
			                    @Override
			                    public void call(String s) {
			                        System.out.println("Rxjava:" + s.length());
			                    }
			                });
			
			
			        for (int i = 0; i < arr.length; i++) {
			            String temp = arr[i];
			            if (temp.startsWith("a")) {
			                temp += " from Alpha";
			                System.out.println("Normal:" + temp.length());
			            }
			
			        }
	
* 由指定的一个 drawable 文件 id 取得图片，并显示在 ImageView 中，并在出现异常的时候打印 Toast 报错：

		    private void simpleDemo() {
		
		        final int drawID = R.mipmap.ic_launcher;
		        Observable
		                .create(new Observable.OnSubscribe<Drawable>() {
		                    @Override
		                    public void call(Subscriber<? super Drawable> subscriber) {
		                        Drawable drawable = getResources().getDrawable(drawID);
		                        subscriber.onNext(drawable);
		                        subscriber.onCompleted();
		                    }
		                })
		                .subscribe(new Observer<Drawable>() {
		                    @Override
		                    public void onCompleted() {
		
		                    }
		
		                    @Override
		                    public void onError(Throwable e) {
		                        Toast.makeText(MainActivity.this, "Error", Toast.LENGTH_SHORT).show();
		                    }
		
		                    @Override
		                    public void onNext(Drawable drawable) {
		                        imageView.setImageDrawable(drawable);
		                    }
		                });
		
		
		    }

6. Scheduler 
		* 作用: 控制线程.指定某一段代码在那个线程里运行.
		* 内置的Scheduler:
			* Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
			* Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
			* Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
			* Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
			* AndroidSchedulers.mainThread(): Android专用,它指定的操作将在 Android 主线程运行。

		* 指定线程的方法:
			* Observable.subscribeOn():指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程
			* Observable.observeOn():指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

	7. 数据变换:
		* 作用: 就是将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列
		* Observable.map:  一对一的转换

			    private void simpleDemo() {
			        Observable
			                .just(R.mipmap.ic_launcher)
			                .map(new Func1<Integer, Drawable>() {
			                    @Override
			                    public Drawable call(Integer integer) {
			                        return getResources().getDrawable(integer);
			                    }
			                })
			                .subscribe(new Action1<Drawable>() {
			                    @Override
			                    public void call(Drawable drawable) {
			                        imageView.setImageDrawable(drawable);
			                    }
			                });
			    }
		* Observable.flatMap: 一对多的转换

				public class Course {
				    private String name;
				    private int id;
				
				    public Course(String name, int id) {
				        this.name = name;
				        this.id = id;
				    }
				
				    public String getName() {
				        return name;
				    }
				
				    public void setName(String name) {
				        this.name = name;
				    }
				
				    public int getId() {
				        return id;
				    }
				
				    public void setId(int id) {
				        this.id = id;
				    }
				}

				public class Student {
				    private String name;
				
				    private ArrayList<Course> courses;
				
				    public Student(String name, ArrayList<Course> courses) {
				        this.name = name;
				        this.courses = courses;
				    }
				
				    public String getName() {
				        return name;
				    }
				
				    public void setName(String name) {
				        this.name = name;
				    }
				
				    public ArrayList<Course> getCourses() {
				        return courses;
				    }
				
				    public void setCourses(ArrayList<Course> courses) {
				        this.courses = courses;
				    }
				}


			    private void student() {
			
			        Course yuwen = new Course("语文", 1);
			        Course shuxue = new Course("数学", 2);
			        Course yingyu = new Course("英文", 3);
			        Course lishi = new Course("历史", 4);
			        Course zhengzhi = new Course("政治", 5);
			        Course xila = new Course("希腊语", 6);
			
			        ArrayList<Course> course1 = new ArrayList<>();
			        course1.add(yuwen);
			        course1.add(shuxue);
			        course1.add(yingyu);
        			course1.add(lishi);
        			course1.add(zhengzhi);
        			course1.add(xila);
			        Student zhangsan = new Student("zhangsan", course1);
			
			        Observable.just(zhangsan)
			                .flatMap(new Func1<Student, Observable<Course>>() {
			                    @Override
			                    public Observable<Course> call(Student student) {
			                        return Observable.from(student.getCourses());
			                    }
			                }).subscribe(new Action1<Course>() {
			            @Override
			            public void call(Course course) {
			                System.out.println(course.getName());
			            }
			        });
			    }

8. 和Retrofit一起使用
		1. 添加依赖: 
			* compile 'com.squareup.retrofit2:retrofit:2.0.1'
			* compile 'com.squareup.retrofit2:converter-gson:2.0.1'
			* compile 'com.squareup.retrofit2:adapter-rxjava:2.0.1'

		2. 利用[http://www.jsonschema2pojo.org/](http://www.jsonschema2pojo.org/)创建数据模型
		3. 创建REST API 接口.注意此时返回的不能是Call而是Observable.示例代码:

				public interface LocationInterface {
					// http://ip.taobao.com/service/getIpInfo.php?ip=202.178.10.23
				    @GET("/service/getIpInfo.php")
				    public Observable<Location> getLocation(@Query("ip") String ip);
				
				}
		4.  创建Retrofit对象,发起请求.注意此时Retrofit需要添加addCallAdapterFactory.示例代码:

		        Retrofit retrofit = new Retrofit.Builder()
		                .baseUrl(BASE2)
		                .addConverterFactory(GsonConverterFactory.create())
		                .addCallAdapterFactory(RxJavaCallAdapterFactory.create())
		                .build();
		
		        LocationInterface locationInterface = retrofit.create(LocationInterface.class);
		        Observable<Location> location = locationInterface.getLocation("8.8.8.8");
		        location
		                .subscribeOn(Schedulers.io())
		                .map(new Func1<Location, String>() {
		
		
		                    @Override
		                    public String call(Location location) {
		                        return location.getData().getCountry();
		                    }
		                })
		                .observeOn(AndroidSchedulers.mainThread())
		                .subscribe(new Action1<String>() {
		                    @Override
		                    public void call(String s) {
		                        textView.setText(s);
		                    }
		                });
		
