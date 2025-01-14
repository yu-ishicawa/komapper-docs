---
title: "クエリの式"
linkTitle: "式"
weight: 200
description: クエリの構成要素を成す式
---

## 概要 {#overview}

本ページは以下の節から成ります。

- [宣言]({{< relref "#declaration" >}})
- [比較演算子]({{< relref "#comparison-operator" >}})
- [論理演算子]({{< relref "#logical-operator" >}})
- [算術演算子]({{< relref "#arithmetic-operator" >}})
- [数学関数]({{< relref "#mathematical-function" >}})
- [文字列関数]({{< relref "#string-function" >}})
- [集約関数]({{< relref "#aggregate-function" >}})
- [条件式]({{< relref "#conditional-expression" >}})
- [スカラサブクエリ]({{< relref "#scalar-subquery" >}})
- [リテラル]({{< relref "#literal" >}})

## 宣言 {#declaration}

Query DSLでは、例えば`where`関数に検索条件を表すラムダ式を渡せます。

```kotlin
QueryDsl.from(a).where { a.addressId eq 1 }
```

KomapperではこのようなSQLの句に対応するようなラムダ式のことを宣言と呼びます。
宣言は全てtypealiasとして`org.komapper.core.dsl.expression`パッケージに定義されています。

Assignment宣言
: VALUES句に相当する`values`関数やSET句に相当する`set`関数が受け取るラムダ式。typealiasは`AssignmentDeclaration`。

Having宣言
: HAVING句に対応する`having`関数が受け取るラムダ式。typealiasは`HavingDeclaration`。

On宣言
: ON句に対応する`on`関数が受け取るラムダ式。typealiasは`OnDeclaration`。

When宣言
: WHEN句に対応する`When`関数が受け取るラムダ式。typealiasは`WhenDeclaration`。

Where宣言
: WHERE句に対応する`where`関数が受け取るラムダ式。typealiasは`WhereDeclaration`。

これらの宣言は下記に示すように合成が可能です。

### plus {#declaration-composition-plus}

`+`演算子を使うと、被演算子の宣言内部に持つ式を順番に実行するような新たな宣言を構築できます。

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
}
val w3: WhereDeclaration = w1 + w2 // +演算子の利用
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? and t0_.VERSION = ?
*/
```

`+`演算子はすべての宣言で利用できます。

### and {#declaration-composition-and}

`and`関数を使うと、宣言を`and`演算子で連結する新たな宣言を構築できます。

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
    or { a.version eq 2 }
}
val w3: WhereDeclaration = w1.and(w2) // and関数の利用
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? and (t0_.VERSION = ? or (t0_.VERSION = ?))
*/
```

`and`関数は、Having、When、Whereの宣言に対して適用できます。

### or {#declaration-composition-or}

`or`関数を使うと、宣言を`or`演算子で連結する新たな宣言を構築できます。

```kotlin
val w1: WhereDeclaration = {
    a.addressId eq 1
}
val w2: WhereDeclaration = {
    a.version eq 1
    a.street eq "STREET 1"
}
val w3: WhereDeclaration = w1.or(w2) // or関数の利用
val query: Query<List<Address>> = QueryDsl.from(a).where(w3)
val list: List<Address> = db.runQuery { query }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ? or (t0_.VERSION = ? and t0_.STREET = ?)
*/
```

`or`関数は、Having、When、Whereの宣言に対して適用できます。


## 比較演算子 {#comparison-operator}

Having、On、When、Whereの [宣言]({{< relref "#declaration" >}}) の中で利用できます。

演算子の引数に`null`を渡した場合その演算子は評価されません。つまりSQLに変換されません。

```kotlin
val nullable: Int? = null
val query = QueryDsl.from(a).where { a.addressId eq nullable }
```

したがって、上記の`query`が実行された場合は次のSQLが発行されます。

```sql
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_
```

### eq {#comparison-operator-eq}

```kotlin
QueryDsl.from(a).where { a.addressId eq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID = ?
*/
```

### notEq {#comparison-operator-noteq}

```kotlin
QueryDsl.from(a).where { a.addressId notEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID <> ?
*/
```

### less {#comparison-operator-less}

```kotlin
QueryDsl.from(a).where { a.addressId less 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID < ?
*/

```

### lessEq {#comparison-operator-lesseq}

```kotlin
QueryDsl.from(a).where { a.addressId lessEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID <= ?
*/
```

### greater {#comparison-operator-greater}

```kotlin
QueryDsl.from(a).where { a.addressId greater 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ?
*/
```

### greaterEq {#comparison-operator-greatereq}

```kotlin
QueryDsl.from(a).where { a.addressId greaterEq 1 }
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID >= ?
*/
```

### isNull {#comparison-operator-isnull}

```kotlin
QueryDsl.from(e).where { e.managerId.isNull() }
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.MANAGER_ID is null
*/
```
### isNotNull {#comparison-operator-isnotnull}

```kotlin
QueryDsl.from(e).where { e.managerId.isNotNull() }
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.MANAGER_ID is not null
*/
```

### like {#comparison-operator-like}

```kotlin
QueryDsl.from(a).where { a.street like "STREET 1_" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notLike {#comparison-operator-notlike}

```kotlin
QueryDsl.from(a).where { a.street notLike "STREET 1_" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### startsWith {#comparison-operator-startswith}

```kotlin
QueryDsl.from(a).where { a.street startsWith "STREET 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notStartsWith {#comparison-operator-notstartswith}

```kotlin
QueryDsl.from(a).where { a.street notStartsWith "STREET 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### contains {#comparison-operator-contains}

```kotlin
QueryDsl.from(a).where { a.street contains "T 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notContains {#comparison-operator-notcontains}

```kotlin
QueryDsl.from(a).where { a.street notContains "T 1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### endsWith {#comparison-operator-endswith}

```kotlin
QueryDsl.from(a).where { a.street endsWith "1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### notEndsWith {#comparison-operator-notendswith}

```kotlin
QueryDsl.from(a).where { a.street notEndsWith "1" }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.STREET not like ? escape ? order by t0_.ADDRESS_ID asc
*/
```

### between {#comparison-operator-between}

```kotlin
QueryDsl.from(a).where { a.addressId between 5..10 }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID between ? and ? order by t0_.ADDRESS_ID asc
*/
```

### notBetween {#comparison-operator-notbetween}

```kotlin
QueryDsl.from(a).where { a.addressId notBetween 5..10 }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID not between ? and ? order by t0_.ADDRESS_ID asc
*/
```

### inList {#comparison-operator-inlist}

```kotlin
QueryDsl.from(a).where { a.addressId inList listOf(9, 10) }.orderBy(a.addressId.desc())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID in (?, ?) order by t0_.ADDRESS_ID desc
*/
```

サブクエリも使えます。

```kotlin
QueryDsl.from(e).where {
    e.addressId inList {
        QueryDsl.from(a)
            .where {
                e.addressId eq a.addressId
                e.employeeName like "%S%"
            }.select(a.addressId)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.ADDRESS_ID in (select t1_.ADDRESS_ID from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notInList {#comparison-operator-notinlist}

```kotlin
QueryDsl.from(a).where { a.addressId notInList (1..9).toList() }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID not in (?, ?, ?, ?, ?, ?, ?, ?, ?) order by t0_.ADDRESS_ID asc
*/
```

サブクエリも使えます。

```kotlin
QueryDsl.from(e).where {
    e.addressId notInList {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }.select(a.addressId)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where t0_.ADDRESS_ID not in (select t1_.ADDRESS_ID from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### inList2 {#comparison-operator-inlist2}

```kotlin
QueryDsl.from(a).where { a.addressId to a.version inList2 listOf(9 to 1, 10 to 1) }.orderBy(a.addressId.desc())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) in ((?, ?), (?, ?)) order by t0_.ADDRESS_ID desc
*/
```

サブクエリも使えます。

```kotlin
QueryDsl.from(e).where {
    e.addressId to e.version inList2 {
        QueryDsl.from(a)
            .where {
                e.addressId eq a.addressId
                e.employeeName like "%S%"
            }.select(a.addressId, a.version)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) in (select t1_.ADDRESS_ID, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notInList2 {#comparison-operator-notinlist2}

```kotlin
QueryDsl.from(a).where { a.addressId to a.version notInList2 listOf(9 to 1, 10 to 1) }.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) not in ((?, ?), (?, ?)) order by t0_.ADDRESS_ID asc
*/
```

サブクエリも使えます。

```kotlin
QueryDsl.from(e).where {
    e.addressId to e.version notInList2 {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }.select(a.addressId, a.version)
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where (t0_.ADDRESS_ID, t0_.VERSION) not in (select t1_.ADDRESS_ID, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### exists {#comparison-operator-exists}

```kotlin
QueryDsl.from(e).where {
    exists {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where exists (select t1_.ADDRESS_ID, t1_.STREET, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

### notExists {#comparison-operator-notexists}

```kotlin
QueryDsl.from(e).where {
    notExists {
        QueryDsl.from(a).where {
            e.addressId eq a.addressId
            e.employeeName like "%S%"
        }
    }
}
/*
select t0_.EMPLOYEE_ID, t0_.EMPLOYEE_NO, t0_.EMPLOYEE_NAME, t0_.MANAGER_ID, t0_.HIREDATE, t0_.SALARY, t0_.DEPARTMENT_ID, t0_.ADDRESS_ID, t0_.VERSION from EMPLOYEE as t0_ where not exists (select t1_.ADDRESS_ID, t1_.STREET, t1_.VERSION from ADDRESS as t1_ where t0_.ADDRESS_ID = t1_.ADDRESS_ID and t0_.EMPLOYEE_NAME like ? escape ?)
*/
```

## 論理演算子 {#logical-operator}

Having、On、When、Whereの [宣言]({{< relref "#declaration" >}}) の中で利用できます。

### and {#logical-operator-and}

宣言の中で式を並べるとAND演算子で連結されます。

```kotlin
QueryDsl.from(a).where {
    a.addressId greater 1
    a.street startsWith "S"
    a.version less 100
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and t0_.STREET like ? escape ? and t0_.VERSION < ?
*/
```

明示的にAND演算子を使いたい場合は`and`関数にラムダ式を渡します。

```kotlin
QueryDsl.from(a).where {
  a.addressId greater 1
  and {
    a.street startsWith "S"
    a.version less 100
  }
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and (t0_.STREET like ? escape ? and t0_.VERSION < ?)
*/
```

### or {#logical-operator-or}

OR演算子で連結したい場合は`or`関数にラムダ式を渡します。

```kotlin
QueryDsl.from(a).where {
  a.addressId greater 1
  or {
    a.street startsWith "S"
    a.version less 100
  }
}
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? or (t0_.STREET like ? escape ? and t0_.VERSION < ?)
*/
```

### not {#logical-operator-not}

NOT演算子を使うには`not`関数にラムダ式を渡します。

```kotlin
QueryDsl.from(a).where {
    a.addressId greater 5
    not {
        a.addressId greaterEq 10
    }
}.orderBy(a.addressId)
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ where t0_.ADDRESS_ID > ? and not (t0_.ADDRESS_ID >= ?) order by t0_.ADDRESS_ID asc
*/
```

## 算術演算子 {#arithmetic-operator}

以下の演算子が使えます。

- `+`
- `-`
- `*`
- `/`
- `%`

これらの演算子は`org.komapper.core.dsl.operator`に定義されています。

`+`演算子を使った例を次に示します。

```kotlin
QueryDsl.update(a).set {
    a.version eq (a.version + 10)
}.where {
    a.addressId eq 1
}
/*
update ADDRESS as t0_ set VERSION = (t0_.VERSION + ?) where t0_.ADDRESS_ID = ?
*/
```

## 数学関数 {#mathematical-function}

次の関数が使えます。

- random

この関数は`org.komapper.core.dsl.operator`に定義されています。

`random`関数を使った例を次に示します。

```kotlin
QueryDsl.from(a).orderBy(random())
/*
select t0_.ADDRESS_ID, t0_.STREET, t0_.VERSION from ADDRESS as t0_ order by random() asc
*/
```

## 文字列関数 {#string-function}

次の関数が使えます。

- concat
- substring
- lower
- upper
- trim
- ltrim
- rtrim

これらの関数は`org.komapper.core.dsl.operator`に定義されています。

`concat`関数を使った例を次に示します。

```kotlin
QueryDsl.update(a).set {
  a.street eq (concat(concat("[", a.street), "]"))
}.where {
  a.addressId eq 1
}
/*
update ADDRESS as t0_ set STREET = (concat((concat(?, t0_.STREET)), ?)) where t0_.ADDRESS_ID = ?
*/
```

## 集約関数 {#aggregate-function}

次の関数が使えます。

- avg
- count
- sum
- max
- min

これらの関数は`org.komapper.core.dsl.operator`に定義されています。

呼び出して得られる式は`having`や`select`で使われることを想定しています。

```kotlin
QueryDsl.from(e)
    .groupBy(e.departmentId)
    .having {
        count(e.employeeId) greaterEq 4L
    }
    .orderBy(e.departmentId)
    .select(e.departmentId, count(e.employeeId))
/*
select t0_.DEPARTMENT_ID, count(t0_.EMPLOYEE_ID) from EMPLOYEE as t0_ group by t0_.DEPARTMENT_ID having count(t0_.EMPLOYEE_ID) >= ? order by t0_.DEPARTMENT_ID asc
*/
```

### avg {#aggregate-function-avg}

```kotlin
QueryDsl.from(a).select(avg(a.addressId))
/*
select avg(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### count {#aggregate-function-count}

SQLの`count(*)`に変換するには`count`関数を引数なしで呼び出します。

```kotlin
QueryDsl.from(a).select(count())
/*
select count(*) from ADDRESS as t0_
*/
```

`count`関数をカラムを指定して呼び出すこともできます。

```kotlin
QueryDsl.from(a).select(count(a.street))
/*
select count(t0_.STREET) from ADDRESS as t0_
*/
```

### sum {#aggregate-function-sum}

```kotlin
QueryDsl.from(a).select(sum(a.addressId))
/*
select sum(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### max {#aggregate-function-max}

```kotlin
QueryDsl.from(a).select(max(a.addressId))
/*
select max(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

### min {#aggregate-function-min}

```kotlin
QueryDsl.from(a).select(min(a.addressId))
/*
select min(t0_.ADDRESS_ID) from ADDRESS as t0_
*/
```

## 条件式 {#conditional-expression}

次の関数や式が使えます。

- coalesce
- case

これらは`org.komapper.core.dsl.operator`に定義されています。

### coalesce {#conditional-expression-coalesce}

`coalesce`関数を使う例です。

```kotlin
QueryDsl.from(a).select(a.addressId, coalesce(a.street, literal("default")))
/*
select t0_.ADDRESS_ID, coalesce(t0_.STREET, 'default') from ADDRESS as t0_
*/
```

### CASE式 {#case-expression}

CASE式を使うには`case`を呼び出します。

```kotlin
val caseExpression = case(
  When(
    {
      a.street eq "STREET 2"
      a.addressId greater 1
    },
    literal("HIT")
  )
) { literal("NO HIT") }
val list: List<Pair<String?, String?>> = db.runQuery {
  QueryDsl.from(a).where { a.addressId inList listOf(1, 2, 3) }
    .orderBy(a.addressId)
    .select(a.street, caseExpression)
}
/*
select t0_.street, case when t0_.street = ? and t0_.address_id > ? then 'HIT' else 'NO HIT' end from address as t0_ where t0_.address_id in (?, ?, ?) order by t0_.address_id asc
*/
```

## スカラサブクエリ {#scalar-subquery}

集約関数を使ってスカラを返すクエリはサブクエリとして他のクエリの`select`関数に渡せます。

```kotlin
val subquery = QueryDsl.from(e).where { d.departmentId eq e.departmentId }.select(count())
val query = QueryDsl.from(d)
    .orderBy(d.departmentId)
    .select(d.departmentName, subquery)
/*
select t0_.department_name, (select count(*) from employee as t1_ where t0_.department_id = t1_.department_id) from department as t0_ order by t0_.department_id asc
*/
```

## リテラル {#literal}

バインド変数を介さず直接値をリテラルとしてSQLに埋め込みたい場合は`literal`を呼び出します。

`literal`関数は`org.komapper.core.dsl.operator`に定義されています。

`literal`関数がサポートする引数の型は以下のものです。

- Boolean
- Int
- Long
- String

使用例です。

```kotlin
QueryDsl.insert(a).values {
  a.addressId eq 100
  a.street eq literal("STREET 100")
  a.version eq literal(100)
}
/*
insert into ADDRESS (ADDRESS_ID, STREET, VERSION) values (?, 'STREET 100', 100)
*/
```