
![EventBus-Publish-Subscribe.png](http://oc1m6u2t9.bkt.clouddn.com/EventBus-Publish-Subscribe.png)

### 简述
[EventBus](https://github.com/greenrobot/EventBus)是一款针对Android优化的发布/订阅事件总线。主要功能是替代Intent,Handler,BroadCast在Fragment，Activity，Service，线程之间传递消息。优点是开销小，代码更优雅，以及将发送者和接收者解耦。

### 基本使用
1.新建一个类，AnyEventType。可以是网络请求返回的字符串，也可以是某个开关状态，也可以是空。

    public class AnyEventType {  
     public AnyEventType(){}  
    }  

2.注册订阅者

    EventBus.getDefault().register(this);

3.发送事件

    EventBus.getDefault().post(new AnyEventType event);

4.编写响应事件订阅方法

    @Subscribe(threadMode = ThreadMode.BACKGROUND, sticky = true, priority = 100)
    public void hello(String str) {
    
    }  

这里说明一下3.0之后可以通过@Subscribe注解,来确定运行的线程threadMode,是否接受粘性事件sticky以及事件优先级priority,而且方法名不在需要onEvent开头,所以又简洁灵活了不少.

5.解除注册

    EventBus.getDefault().unregister(this);

### 源码解析
##### 1.新建EventBus
- 默认可通过静态函数 getDefault 获取单例

      public static EventBus getDefault() {
        if (defaultInstance == null) {
          synchronized (EventBus.class) {
        if (defaultInstance == null) {
          defaultInstance = new EventBus();
        }}}
      return defaultInstance;
      }

- EventBusBuilder 新建一个 EventBus

      public static EventBusBuilder builder() {
      return new EventBusBuilder();
      }

- 构造函数新建一个EventBus

      public EventBus() {
        this(DEFAULT_BUILDER);
      }

##### 2.register
register 函数中会先根据订阅者类名去subscriberMethodFinder
中查找当前订阅者所有事件响应函数，然后循环每一个事件响应函数，依次执行subscribe 函数

    public void register(Object subscriber) { 
    subscriberClass = subscriber.getClass();
    List<SubscriberMethod> subscriberMethods = subscriberMethodFinder.findSubscriberMethods(subscriberClass); 
    synchronized (this) { 
      for (SubscriberMethod subscriberMethod : subscriberMethods) { 
        subscribe(subscriber, subscriberMethod); 
        } 
      } 
    }

![register.png](http://oc1m6u2t9.bkt.clouddn.com/register.png)

##### 3.subscribe
源码太长就不全部贴出来了

1.首先通过subscriptionsByEventType得到该事件类型所有订阅者信息队列，根据优先级将当前订阅者信息插入到订阅者队列subscriptionsByEventType中；如果添加过就抛出异常。

    CopyOnWriteArrayList<Subscription> subscriptions = subscriptionsByEventType.get(eventType);
    if (subscriptions == null) {
        subscriptions = new CopyOnWriteArrayList<>();
        subscriptionsByEventType.put(eventType, subscriptions);
      } else {
      if (subscriptions.contains(newSubscription)) {
          throw new EventBusException("Subscriber " + subscriber.getClass() + " already registered to event "+ eventType);
        }
      }

2.在typesBySubscriber中得到当前订阅者订阅的所有事件队列，将此事件保存到队列typesBySubscriber中，用于后续取消订阅；

    List<Class<?>> subscribedEvents = typesBySubscriber.get(subscriber); 
    if (subscribedEvents == null) { 
        subscribedEvents = new ArrayList<>();         
        typesBySubscriber.put(subscriber, subscribedEvents); 
      }

3.检查这个事件是否是 Sticky 事件，如果是则立即分发sticky事件

    if (subscriberMethod.sticky) { 
    //eventInheritance 表示是否分发订阅了响应事件类父类事件的方法 
        if (eventInheritance) { 
           Set<Map.Entry<Class<?>, Object>> entries = stickyEvents.entrySet(); 
           for (Map.Entry<Class<?>, Object> entry : entries) { 
              Class<?> candidateEventType = entry.getKey(); 
              if (eventType.isAssignableFrom(candidateEventType)) { 
                Object stickyEvent = entry.getValue(); 
                checkPostStickyEventToSubscription(newSubscription, stickyEvent); 
               } 
            } 
          } else { 
            Object stickyEvent = stickyEvents.get(eventType); 
            checkPostStickyEventToSubscription(newSubscription, stickyEvent); 
          } 
    }

##### 4.post
首先得到当前线程的 post 信息PostingThreadState，其中包含事件队列，将当前事件添加到其事件队列中，然后循环调用postSingleEvent 函数发布队列中的每个事件。



    public void post(Object event) { 
    PostingThreadState postingState = currentPostingThreadState.get(); 
    List<Object> eventQueue = postingState.eventQueue; eventQueue.add(event); 
    if (!postingState.isPosting) { 
        postingState.isMainThread = Looper.getMainLooper() == Looper.myLooper(); 
        postingState.isPosting = true; 
        if (postingState.canceled) { 
          throw new EventBusException("Internal error. Abort state was not reset"); 
        } try {
        while (!eventQueue.isEmpty()) { 
        postSingleEvent(eventQueue.remove(0), postingState); 
        } 
      } finally { 
        postingState.isPosting = false; 
        postingState.isMainThread = false; 
        } 
      } 
    }

postToSubscription 函数中会判断订阅者的 ThreadMode，从而决定在什么 Mode 下执行事件响应函数。ThreadMode 共有四类：
- PostThread：默认的 ThreadMode，表示在执行 Post 操作的线程直接调用订阅者的事件响应方法，不论该线程是否为主线程（UI 线程）。当该线程为主线程时，响应方法中不能有耗时操作，否则有卡主线程的风险。适用场景：**对于是否在主线程执行无要求，但若 Post 线程为主线程，不能耗时的操作**；
- MainThread：在主线程中执行响应方法。如果发布线程就是主线程，则直接调用订阅者的事件响应方法，否则通过主线程的 Handler 发送消息在主线程中处理——调用订阅者的事件响应函数。显然，MainThread类的方法也不能有耗时操作，以避免卡主线程。适用场景：**必须在主线程执行的操作**；
- BackgroundThread：在后台线程中执行响应方法。如果发布线程**不是**主线程，则直接调用订阅者的事件响应函数，否则启动**唯一的**后台线程去处理。由于后台线程是唯一的，当事件超过一个的时候，它们会被放在队列中依次执行，因此该类响应方法虽然没有PostThread类和MainThread类方法对性能敏感，但最好不要有重度耗时的操作或太频繁的轻度耗时操作，以造成其他操作等待。适用场景：*操作轻微耗时且不会过于频繁*，即一般的耗时操作都可以放在这里；
- Async：不论发布线程是否为主线程，都使用一个空闲线程来处理。和BackgroundThread不同的是，Async类的所有线程是相互独立的，因此不会出现卡线程的问题。适用场景：*长耗时操作，例如网络访问*。


![post.png](http://oc1m6u2t9.bkt.clouddn.com/post.png)



##### 5.unregister
通过typesBySubscriber来取出这个subscriber订阅者订阅的事件类型,从typesBySubscriber移除subscriber。

    public synchronized void unregister(Object subscriber) { 
    List<Class<?>> subscribedTypes = typesBySubscriber.get(subscriber); 
    if (subscribedTypes != null) { 
        for (Class<?> eventType : subscribedTypes) { 
            unsubscribeByEventType(subscriber, eventType); 
        } 
      typesBySubscriber.remove(subscriber); 
    } else { 
        Log.w(TAG, "Subscriber to unregister was not registered before: " + subscriber.getClass()); 
      } 
    }
