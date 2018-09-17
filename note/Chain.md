- [介绍](#%E4%BB%8B%E7%BB%8D)
- [待处理对象](#%E5%BE%85%E5%A4%84%E7%90%86%E5%AF%B9%E8%B1%A1)
- [抽象处理类](#%E6%8A%BD%E8%B1%A1%E5%A4%84%E7%90%86%E7%B1%BB)
- [具体处理类](#%E5%85%B7%E4%BD%93%E5%A4%84%E7%90%86%E7%B1%BB)
- [客户端](#%E5%AE%A2%E6%88%B7%E7%AB%AF)
### 介绍
- 属于行为型设计模式
- 优点：实现了请求者与处理者的代码分离，可以自由组合工作流程，类与类直接松耦合
- 缺点：以链的形式在对象间传递消息，可能会影响处理的速度
- 责任链模式应用在 Spring MVC 的初始化过程中
  
> 今天通过一个面试流程来讲解一下责任链模式
> 
### 待处理对象
```java
public class Applicant {

    private Interviewer interviewer;

    private int score;

    public int getScore() {
        return score;
    }

    public void setScore(int score) {
        this.score = score;
    }

    public void setInterviewer(Interviewer interviewer) {
        this.interviewer = interviewer;
    }

    public Interviewer getInterviewer() {
        return interviewer;
    }
}
```

### 抽象处理类
```java
public abstract class Interviewer {

    public String name;

    Interviewer next;

    public Interviewer(String name) {
        this.name = name;
    }

    public void setNext(Interviewer interviewer) {
        this.next = interviewer;
    }

    public abstract void handleApplicant(Applicant applicant);
}
```

### 具体处理类
```java
// 初试
public class First extends Interviewer {

    public First(String name) {
        super(name);
    }

    @Override
    public void handleApplicant(Applicant applicant) {
        System.out.println("I am " + name + " interviewer");
        if (applicant.getScore() < 5) {
            System.out.println("The First Interview failed");
        } else {
            System.out.println("Enter the next interview");
            if (this.next != null) {
                this.next.handleApplicant(applicant);
            }
        }
    }
}
```

```java
// 复试
public class Second extends Interviewer {
    
    public Second(String name) {
        super(name);
    }

    @Override
    public void handleApplicant(Applicant applicant) {
        System.out.println("I am " + name + " interviewer");
        if (applicant.getScore() < 5) {
            System.out.println("The Second Interview failed");
        } else {
            System.out.println("You've already passed all the interviews");
        }
    }
}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        Applicant applicant = new Applicant();
        applicant.setScore(7);
        Interviewer interviewer1 = new First("stalary");
        Interviewer interviewer2 = new Second("claire");
        interviewer1.setNext(interviewer2);
        interviewer1.handleApplicant(applicant);
        System.out.println("-------------------");
        Applicant loser = new Applicant();
        loser.setScore(1);
        interviewer1.handleApplicant(loser);
    }
}
```

```java
// 测试结果
I am stalary interviewer
Enter the next interview
I am claire interviewer
You've already passed all the interviews
-------------------
I am stalary interviewer
The First Interview failed
```