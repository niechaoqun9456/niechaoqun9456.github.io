---
layout: post
title:  "Custom-BindingAdapter的经验与教训"
date:   2020-04-15 09:02:10 +0800
categories: jekyll update
---
### 尽量使用SDK内提供的BindingAdapter
- 过多的Custom BindingAdapter不好组织和维护
- BindingAdapter不便于测试
- 自定义BindingAdapter通常没有优化

### 优化你的自定义BindingAdapter
可以参考SDK内的BindingAdapter实现，通常需要对比oldValue和newValeu，在必要时刷新控件，避免重复layout

### 拆分ViewState
避免使用一个大对象（ObservableField or LiveData）数据绑定，这样会造成不必要的刷新。例如一个用户资料界面，分别显示姓名、邮箱、性别等，只绑定一个LiveData<User>对象的话，当你更新用户名时，邮箱、性别对应的view也会刷新。

### Reference
[Data Binding — Lessons Learnt](https://medium.com/androiddevelopers/data-binding-lessons-learnt-4fd16576b719)