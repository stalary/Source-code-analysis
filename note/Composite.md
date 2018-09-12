### 介绍
- 组合模式属于结构型设计模式
- 优点：简化客户端代码，符合开闭原则，客户端无需因为加入了新的组件而改变代码
- 缺点：不容易限制组合中的组件

### 组件
```java
public abstract class Company {

    private String name;

    public Company(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public abstract void addCompany(Company company);

    public abstract void deleteCompany(Company company);

    public abstract void show();

    @Override
    public String toString() {
        return "Company{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

### 叶子
```java
// 产品部
public class ProductDepartment extends Company {

    public ProductDepartment(String name) {
        super(name);
    }

    @Override
    public void addCompany(Company company) {

    }

    @Override
    public void deleteCompany(Company company) {

    }

    @Override
    public void show() {
        System.out.println(this.getName());
    }
}
```

```java
// 技术部
public class TechnologyDepartment extends Company {
    public TechnologyDepartment(String name) {
        super(name);
    }

    @Override
    public void addCompany(Company company) {

    }

    @Override
    public void deleteCompany(Company company) {

    }

    @Override
    public void show() {
        System.out.println(this.getName());
    }
}
```

### 根
```java
public class BranchCompany extends Company {

    private List<Company> departments;

    public BranchCompany(String name) {
        super(name);
        departments = new ArrayList<>();
    }

    @Override
    public void addCompany(Company company) {
        departments.add(company);
    }

    @Override
    public void deleteCompany(Company company) {
        departments.remove(company);
    }

    @Override
    public void show() {
        System.out.println(departments);
    }

}
```

### 客户端
```java
public class Main {

    public static void main(String[] args) {
        Company hangZhouCompany = new BranchCompany("网易杭州");
        hangZhouCompany.addCompany(new ProductDepartment("网易杭州-产品部"));
        hangZhouCompany.addCompany(new TechnologyDepartment("网易杭州-技术部"));
        Company beiJingCompany = new BranchCompany("网易北京");
        beiJingCompany.addCompany(new ProductDepartment("网易北京-产品部"));
        beiJingCompany.addCompany(new TechnologyDepartment("网易北京-技术部"));
        hangZhouCompany.show();
        beiJingCompany.show();
    }
}
```

```java
// 测试结果
[Company{name='网易杭州-产品部'}, Company{name='网易杭州-技术部'}]
[Company{name='网易北京-产品部'}, Company{name='网易北京-技术部'}]
```