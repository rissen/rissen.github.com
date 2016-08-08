---
layout: posts
title: "Oracle NUMBER(precision,scale) and Hana DECIMAL(precision,scale) difference"
description: "Oracle NUMBER(precision,scale) and Hana DECIMAL(precision,scale) difference"
category: others
tags: [oracle,hana]
---

# Oracle NUMBER(precision,scale) and Hana DECIMAL(precision,scale) difference


公司的产品同时支持Oracle和Hana，每次测试都需要覆盖到这两个DB。最近发现一个很诡异的bug，同一个测试用例，在Oracle上能通过，可是在Hana上却失败了，测试点是比较一个字段的值，测试log显示Hana上的值和Oracle的值相差1。debug了code发现数据在保存到DB之前没有在代码层面进行四舍五入，而是直接利用了database自带的四舍五入，对这种做法持保留意见。

查看了Oracle和Hana的文档，发现还是有较大区别的。Oracle的NUMBER类型具备四舍五入功能，譬如某个字段定义为NUMBER(38,0)，当insert/update的值是66.2时，实际保存在db的是66，当insert/update的值是66.5时，实际保存在db的是61。但是Hana的DECIMAL(precision,scale)却不具备这个功能，不管是66.2还是66.5，保存在db的都是66，超出精度的值直接被剪掉。如果恰好应用程序没有在代码层做四舍五入而是用了Oracle自带的四舍五入，当迁移到Hana之后数据就不再准确了。如果把database从Oracle迁移到Hana一定要注意这个问题。


## 参考链接
[Overview of Numeric Datatypes](https://docs.oracle.com/cd/B28359_01/server.111/b28318/datatype.htm#i22289)

[Hana Numeric Types](https://help.sap.com/saphelp_hanaplatform/helpdata/en/20/a1569875191014b507cf392724b7eb/content.htm#loio20a1569875191014b507cf392724b7eb___csql_data_types_1sql_data_types_introduction_numeric)