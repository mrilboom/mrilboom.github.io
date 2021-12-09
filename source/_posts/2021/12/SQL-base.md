---
tags:
  - Oracle
mathjax: true
comments: false
hidden: false
date: 2021-12-09 14:16:10
title: 基本的SQL语法

---

***

这里记录了一些非常基本的增删改查语法，由于现在为止其实用的不多，每次用到都得回忆查一遍，查出来也是各种各样都有，需要花费时间剔除很多无效信息，不如直接列出来以便查阅。<!-- more -->

***

## INSERT

INSERT INTO `<TABLE>`[(`<COL1>`,`<COL2>`,...`<COLn>`)] VALUES(`<V1>`,`<V2>`...`<Vn>`)

INSERT INTO `<TABLE>`[(`<COL1>`,`<COL2>`,...`<COLn>`)] (SUB QUERY)

## DELETE

DELETE FROM `<TABLE>` WHERE `<CONDITION>`

## UPDATE

UPDATE `<TABLE>` SET `<COL1>`=`<V1>`,`<COL2>`=`<V2>`...`<COLn>`=`<Vn>`  WHERE `<CONDITION>`

## SELECT

SELECT `<COL1>`,`<COL2>`...`<COLn>` FROM `<TABLE>` WHERE `<CONDITION>`