# enum枚举类型

## 1.概览

通过枚举类类型避免了定义常量，将所有和pizza订单的状态的常量都统一放到了一个枚举类型里面

## 2.自定义枚举方法

```java
public class Pizza {
    private PizzaStatus status;
    public enum PizzaStatus{
        ORDERED,
        READY,
        DELIVERED;
    }
    public PizzaStatus getStatus() {
        return status;
    }
    public boolean isDeliverable(){
        return getStatus() == PizzaStatus.READY;
    }
}
```

## 3.使用==比较枚举类型

枚举类型确保JVM中仅存在一个常量实例，因此可以安全的使用==运算符比较两个变量，此外，==元素安抚可提供编译和运行时的安全性。相反，使用equals方法会跑出NullPointerException

## 4.在Switch语句中使用枚举类型

```java
public int getDeliverTimeInDays(){
    switch (status){
        case ORDERED:
            return 5;
        case READY:
            return 2;
        case DELIVERED:
            return 0;
    }
    return 0;
}
```

## 5.枚举类型的属性，方法和构造函数

```java
public class Pizza {

    private PizzaStatus status;

    public PizzaStatus getStatus() {
        return status;
    }

    public void setStatus(PizzaStatus status) {
        this.status = status;
    }

    public enum PizzaStatus {
        ORDERED (5){
            @Override
            public boolean isOrdered() {
                return true;
            }
        },
        READY (2){
            @Override
            public boolean isReady() {
                return true;
            }
        },
        DELIVERED (0){
            @Override
            public boolean isDelivered() {
                return true;
            }
        };

        private int timeToDelivery;

        public boolean isOrdered() {return false;}

        public boolean isReady() {return false;}

        public boolean isDelivered(){return false;}

        public int getTimeToDelivery() {
            return timeToDelivery;
        }

        PizzaStatus (int timeToDelivery) {
            this.timeToDelivery = timeToDelivery;
        }
    }

    public boolean isDeliverable() {
        return this.status.isReady();
    }

    public void printTimeToDeliver() {
        System.out.println("Time to delivery is " +
                           this.getStatus().getTimeToDelivery());
    }

    // Methods that set and get the status variable.
}
```

## 6.EnumSet and EnumMap

### 6.1 Enumset

`Enumset` 是一种专门为枚举类型所设计的set类型 与`HashSet `相比，由于使用了内部向量表示，因此它是特定`Enum`常量集的非常有效且紧凑的表达形式。它提供了类型安全的替代方法，一替代传统的基于int的标志位，使我们能够编写更易读和易于维护的简洁代码。



