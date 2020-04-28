---
title: "Python3からSQLiteを使う"
slug: "sqlite-from-python3"
subtitle:    ""
description: ""
date:        "2020-05-28"
author:      "Masahiro Hiramori"
image:       ""
tags:        ["Python", "SQLite"]
categories:  ["Tech" ]
draft:       false
---

Python 3にはSQLiteを扱うためのモジュールが付属している。
サンプルコードを書いたのでメモ。

# step-by-stepで説明

```python3
dbname = "test.db"
conn = sqlite3.connect(dbname)
```
データベースへ接続する。

```python3
cur = conn.cursor()
```
データベースを操作するカーソルオブジェクトを取得。


```python3
cur.execute('DROP TABLE IF EXISTS sample')
cur.execute('CREATE TABLE IF NOT EXISTS sample (sensor1 real, sensor2 real, sensor3 real)')
```
カーソルオブジェクトに対して`.execute()`メソッドでSQL文を実行する。
テーブル`sample`が存在していれば、削除する。
新たなテーブル`sample`を作成する。

```python3
for i in range(24*60*60):
    sensor1 = np.random.normal()
    sensor2 = np.random.normal()
    sensor3 = np.random.normal()
    cur.execute('INSERT INTO sample VALUES(?, ?, ?)', (sensor1, sensor2, sensor3))
```
NumPyで3つのランダムな実数を生成し、テーブルへレコードをINSERTする。

```python3
conn.commit()
```
データベースへコミットし、変更を反映させる。

```python3
cur.execute('SELECT * FROM sample')
for i in range(5):
    print(cur.fetchone())
```
テーブル`sample`からレコードを取得し、5件表示する。

```python3
conn.close()
```
接続を閉じる。

# 全体ソースコード

```python3
import numpy as np
import sqlite3

np.random.seed(seed=0)

dbname = "test.db"
conn = sqlite3.connect(dbname)

cur = conn.cursor()

cur.execute('DROP TABLE IF EXISTS sample')
cur.execute('CREATE TABLE IF NOT EXISTS sample (sensor1 real, sensor2 real, sensor3 real)')

for i in range(24*60*60):
    sensor1 = np.random.normal()
    sensor2 = np.random.normal()
    sensor3 = np.random.normal()
    cur.execute('INSERT INTO sample VALUES(?, ?, ?)', (sensor1, sensor2, sensor3))

conn.commit()

cur.execute('SELECT * FROM sample')
for i in range(5):
    print(cur.fetchone())

conn.close()
```
