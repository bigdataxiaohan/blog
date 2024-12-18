---
title: Flink复杂事件处理
date: 2020-11-19 10:30:41
tags: Flink
categories: 大数据
---

## 简介

复杂事件处理（Complex Event Process，简称CEP）用来检测无尽数据流中的复杂模式，拥有从不同的数据行中辨识查找模式的能力。模式匹配是复杂事件处理的一个强大援助。 包括受一系列事件驱动的各种业务流程，例如在安全应用中侦测异常行为；在金融应用中查找价格、交易量和其他行为的模式。其他常见的用途如欺诈检测应用和传感器数据的分析等。

## 概念

### 简单事件

简单时间存在于现实生活中，其特点为处理单一事件，事件的定义可以直接观察出来，无需关注多个事件的关系，通过简单的数据处理即可得出结果。

### 复杂事件

由多个事件组成的复合事件。复杂事件处理监测分析事件流，当特定事件发生时来触发某些动作。

复杂事件中事件与事件之间包含多种类型关系，常见的有时序关系、聚合关系、层次关系、依赖关系及因果关系等。

## 开发步骤

FlinkCEP中提供了Pattern API 用于对输入流数据的复杂事件**规则定义**，并从事件流中抽取事件结果。包含四个步骤：

- 输入事件流的创建
- Pattern 的定义
- Pattern 应用在事件流上检测
- 选取结果

