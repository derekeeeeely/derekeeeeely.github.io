---
title: node/V8小笔记-2
abbrlink: 33e90ba4
date: 2018-01-06 16:21:12
tags:
---

&emsp;&emsp;node/V8学习笔记系列2

<!-- more -->

### js与C++交互

#### 模板类

V8中有两个模板类：
  - 对象模板 (ObjectTemplate)
  - 函数模板 (FunctionTemplate)

#### js调用C++

  - 变量
  ```c++
  // Create NameGetter and NameSetter
  static Handle<Value> NameGetter(Local<String> name, const AccessorInfo& info) {
    return String::New((char*)&sname,strlen((char*)&sname));
  }
  static void NameSetter(Local<String> name, Local<Value> value, const AccessorInfo& info) {
    Local<String> str = value->ToString();
    str->WriteAscii((char*)&sname);
  }
  // 在main函数内，将上述setter注册到global
  // Create a template for the global object.
  Handle<ObjectTemplate> global = ObjectTemplate::New();
  //public the name variable to script
  global->SetAccessor(String::New("name"), NameGetter, NameSetter);
  ```
  - 函数
  ```c++
  // 编写函数
  Handle<Value> func(const Arguments& args){//return something}
  // 公开给脚本
  global->Set(String::New("func"),FunctionTemplate::New(func))
  ```
  - 类
    定义类
    定义构造函数
    包装getter、setter
    创建函数模板，将其与字符串‘Person’绑定，set到global中

    ```c++
    Handle<FunctionTemplate> person_template = FunctionTemplate::New(PersonConstructor);
    person_template->SetClassName(String::New("Person"));
    global->Set(String::New("Person"), person_template);
    ```
    定义原型模板
    ```c++
    Handle<ObjectTemplate> person_proto = person_template->PrototypeTemplate();
    person_proto->Set("getAge", FunctionTemplate::New(PersonGetAge));
    person_proto->Set("setAge", FunctionTemplate::New(PersonSetAge));
    person_proto->Set("getName", FunctionTemplate::New(PersonGetName));
    person_proto->Set("setName", FunctionTemplate::New(PersonSetName));
    ```
    设置实例模板
    ```c++
    Handle<ObjectTemplate> person_inst = person_template->InstanceTemplate();
    person_inst->SetInternalFieldCount(1);
    ```

