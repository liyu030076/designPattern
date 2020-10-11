通知变化 给 多个类 的方法

#1. 目标

1.  构建 object 间1对多 的依赖关系，使得当 **`主object`** 改变状态时，**`依赖objects`** 都自动被 通知notify和更新update

2. **`把主object 封装在 Subject 抽象中, 把依赖objects 封装在  Observer 体系 hierarchy 中`**

3. **`MVC（Model-View-Controller）的 View 部分`**

Smalltalk 中 引入了 MVC 技术，以方便 交互式 软件开发

#2. 讨论

1. 数据模型/业务逻辑 用 Subject 维持

Subject 与 Observers 解耦，把 View 功能 delegate 给 各具体的 Observer

解耦 发送者 和 接收者

2. 有待考虑的问题

（1）Subject 发生 一连串consecutive 变化时，如何仅把 某个变化 通过给 Observer
（2）1个 Observer 观察 多个 Subject
（3）Subject 通知 Observers 自己 何时消失

#3. 注意：

1. client 配置 Observer 数量和类型
2. Observer 注册自己到 Subject
3. **`Subject 只与 Observer Base类 耦合: (code 中4.1)`**

4. Subject 可以 Push 信息给 Observers
Observers 可以从 Subject 中 Pull 信息

Subject 调用 Observers，Observers 也可以 回调 Subject

#4. structure & code

![Observer.jpg](https://upload-images.jianshu.io/upload_images/20172887-a98f16ff8c741724.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4.1 用Observer抽象类 挂接

**`类A的成员函数 通过 所拥有的类B的指针 调类B及其子类的成员函数, 则类A 的成员函数定义 必须放在 类体外， 且 放在 类B定义（而不能只是声明）后`**

```
#include "stdafx.h"
#include <iostream>
#include <vector>

using namespace std;

class Subject
{
private:
    //1. 必须 写成 < class Observer* >
    //相当于 用语句 class Observer; 来提前声明 Observer
    vector < class Observer* > views;
    int value;
public:
	//2.3 Subject 的 挂接函数 
    void attch(Observer* pObserver) 
    {
        views.push_back(pObserver);
    }

	//3.1 主object 状态变化时, 通知 依赖objects
    void setValAndnotify(int val)
    {
        value = val;
        notify();
    }
    int getVal() { return value; }

	//3.2 Subject 通知函数 notify() 放在 Observer基类后定义, 
	// 因为要通过Observer指针, 间接调Observers的更新函数update()
    void notify();
};

class Observer
{
public:
    Observer(Subject* pSubject)
    {
        //2.2 抽象Observer 挂接
		pSubject->attch(this); 
    }
    virtual void update(Subject* pSubject) = 0;
};

void Subject::notify()
{
    //3.3 Subject Push 信息 : notify Observers update 观察到的 Subject 状态
    for(int i = 0; i < views.size(); ++i)   
        views[i]->update(this);
}

class Observer1:public Observer
{
private:
	int valObserver1;
public:
	// 2.1 具体Observer 构造函数: 
	//(1)从 client 接收 Subject*
	//(2)调抽象Observer 构造函数: 
	// 用 Subject* 调挂接函数, 把Observers对象指针挂到Subject的vector中
    Observer1(Subject* pSubject): valObserver1(0), Observer(pSubject)
    { 
        //2.4 Observers Pull 信息
        cout << pSubject->getVal() << endl;
    }

	//3.4 Observers update 
    void update(Subject* pSubject)
    {
        cout << "Observer1-curve: " << pSubject->getVal() << endl;
    }
};
class Observer2:public Observer
{
private:
	int valObserver2;
public:
    Observer2(Subject* pSubject): valObserver2(0), Observer(pSubject){ }
    void update(Subject* pSubject)
    {
        cout << "Observer2-graph: " << pSubject->getVal() << endl;
    }
};
int main()
{
    //client
	//(1)创建1个 Subject 对象
	Subject subject;

	//(2)构造  Observers对象: 用Subject对象指针
    Observer1 observer1(&subject);
    Observer2 observer2(&subject);

	//(3)Sunject 状态变化, 并通知 Observers 更新
    subject.setValAndnotify(2);
}
```
2个难点：

**`用Observer抽象类 挂接时, 挂接的指针 对应的内存 / 对象component 是 Observer`**
![image.png](https://upload-images.jianshu.io/upload_images/20172887-6fae51657f595e04.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**通知 某个ConcreteObserver时, 挂接的指针 对应的内存 / 对象component `变成了` 该ConcreteObserver**
![image.png](https://upload-images.jianshu.io/upload_images/20172887-12c657c81a06d851.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


##4.2 用 Observer具体类 挂接 : 更清晰, 更好理解

```
#include "stdafx.h"
#include <iostream>
#include <vector>

using namespace std;

class Subject
{
private:
    //1. 必须 写成 < class Observer* >
    //相当于 用语句 class Observer; 来提前声明 Observer
    vector < class Observer* > views;
    int value;
public:
	//2.3 Subject 的 挂接函数 
    void attch(Observer* pObserver) 
    {
        views.push_back(pObserver);
    }

	//3.1 主object 状态变化时, 通知 依赖objects
    void setValAndnotify(int val)
    {
        value = val;
        notify();
    }
    int getVal() { return value; }

	//3.2 Subject 通知函数 notify() 放在 Observer基类后定义, 
	// 因为要通过Observer指针, 间接调Observers的更新函数update()
    void notify();
};

class Observer
{
public:
    virtual void update(Subject* pSubject) = 0;
};

void Subject::notify()
{
    //3.3 Subject Push 信息 : notify Observers update 观察到的 Subject 状态
    for(int i = 0; i < views.size(); ++i)   
        views[i]->update(this);
}

class Observer1:public Observer
{
private:
	int valObserver1;
public:
	// 2.1 具体Observer 构造函数: 
	//(1)从 client 接收 Subject*
	//(2)用 Subject* 调挂接函数, 把 Observer具体类对象指针 挂到Subject的vector中
    Observer1(Subject* pSubject): valObserver1(0)
    { 
		//2.2 Observer具体类 挂接
		pSubject->attch(this);

        //2.4 Observers Pull 信息
        cout << pSubject->getVal() << endl;
    }

	//3.4 Observers update 
    void update(Subject* pSubject)
    {
        cout << "Observer1-curve: " << pSubject->getVal() << endl;
    }
};
class Observer2:public Observer
{
private:
	int valObserver2;
public:
    Observer2(Subject* pSubject): valObserver2(0)
	{ 
		pSubject->attch(this);
	}
    void update(Subject* pSubject)
    {
		
        cout << "Observer2-graph: " << pSubject->getVal() << endl;
    }
};
int main()
{
    //client
	//(1)创建1个 Subject 对象
	Subject subject;

	//(2)构造  Observers对象: 用Subject对象指针
    Observer1 observer1(&subject);
    Observer2 observer2(&subject);

	//(3)Sunject 状态变化, 并通知 Observers 更新
    subject.setValAndnotify(2);
}
```

![image.png](https://upload-images.jianshu.io/upload_images/20172887-27e103a65c0e1f8d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/20172887-37813cd833a3ee84.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
