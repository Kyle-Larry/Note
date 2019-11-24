#### Spring @EventListener 事件与监听
1. 自定义Application Event
    ```
    // 继承ApplicationEvent
    public class MyEvent extends ApplicationEvent {
        private String action; // onAdd,onUpdate,onDelete
        ...
        ...
        ...
    }
1. 自定义监听
    ```
    @Component
    public class MyListener {
        @EventListener(condition = "#event.action =='onAdd'")
        public void handleDemoEvent(MyEvent event) {
            logger.info("发布的data为:{}", event.getData());
        }
    }

1. 发布Event
    ```
    @Autowired
    ApplicationEventPublisher eventPublisher;
            
    eventPublisher.publishEvent(new MyEvent(this,"onAdd"));
