# SQL Data Definition

The set of relations in a database must be specified to the system by means of a data-definition language (DDL).
## Basic Types
* char(n): A fixed-length character string with user-specified length n. The full form, character, can be used instead.
* varchar(n): A variable-length character string with user-specified maximum length n. The full form, character varying, is equivalent.
* int: An integer (a finite subset of the integers that is machine dependent). The full form, integer, is equivalent.
* smallint: A small integer (a machine-dependent subset of the integer type).
* numeric( p, d): A fixed-point number with user-specified precision. The number consists of p digits (plus a sign), and d of the p digits are to the right of the decimal point. Thus, numeric(3,1) allows 44.5 to be stored exactly, but neither 444.5 or 0.32 can be stored exactly in a field of this type.
* real, double precision: Floating-point and double-precision floating-point numbers with machine-dependent precision.
* float(n): A floating-point number, with precision of at least n digits.

Each type may include a special value called the null value.

## Basic Schema Definition
We define an SQL relation by using the __create table__ command.
```SQL
create table department
(dept_name varchar (20),
building varchar (15),
budget numeric (12,2),
primary key (dept_name));
```
SQL supports a number of different integrity constraints. SQL prevents any update to the database that violates an integrity constraint. In this section, we discuss only a few of them:
* __primary key (A1, A2, ..., An)__: The primary-key specification says that attributes A1, A2, ..., An form the primary key for the relation. The primary-key attributes are required to be nonnull and unique; that is, no tuple can have a null value for a primary-key attribute, and no two tuples in the relation can be equal on all the primary-key attributes. Although the primary-key specification is optional, it is generally a good idea to specify a primary key for each relation.
* __foreign key (A1, A2, ..., An) references s__: The foreign key specification says that the values of attributes (A1, A2, ..., An) for any tuple in the relation must correspond to values of the primary key attributes of some tuple in relation s.
* __not null__: The not null constraint on an attribute specifies that the null value is not allowed for that attribute; in other words, the constraint excludes the null value from the domain of that attribute.

A newly created relation is empty initially. We can use the __insert__ command to load data into the relation.
```SQL
insert into instructor
values (10211, 'Smith', 'Biology', 66000);
```
We can use the _delete_ command to delete tuples from a relation.
```SQL
delete from student;
```
To remove a relation from an SQL database, we use the __drop table__ command. The drop table command deletes all information about the dropped relation from the database.

```SQL
drop table r;
```

We use the __alter table__ command to add attributes to an existing relation. All tuples in the relation are assigned null as the value for the new attribute. The form of the alter table command is
```SQL
alter table r add A D;
```
where r is the name of an existing relation, A is the name of the attribute to be added, and D is the type of the added attribute. We can drop attributes from a relation by the command
```SQL
alter table r drop A;
```
where r is the name of an existing relation, and A is the name of an attribute of the relation.

# Basic Structure of SQL Queries

## Queries on a Single Relation
Consider another query, "Find the department names of all instructors," which can be written as:
```SQL
select dept_name from instructor;
```
Since more than one instructor can belong to a department, a department name could appear more than once in the instructor relation. In practice, duplicate elimination is time-consuming. Therefore, SQL allows duplicates in relations as well as in the results of SQL expressions. Thus, the preceding SQL query lists each department name once for every tuple in which it appears in the instructor relation. In those cases where we want to force the elimination of duplicates, we insert the keyword __distinct__ after select. We can rewrite the preceding query as:
```SQL
select distinct dept_name from instructor;
```

SQL allows us to use the keyword __all__ to specify explicitly that duplicates are not removed. Since duplicate retention is the default, we shall not use all in our examples.  

The __select__ clause may also contain arithmetic expressions involving the operators +, −, *, and / operating on constants or attributes of tuples. For example, the query: 
```SQL
select ID , name, dept name, salary * 1.1
from instructor;
```
returns a relation that is the same as the instructor relation, except that the attribute salary is multiplied by 1.1.  

The __where__ clause allows us to select only those rows in the result relation of the from clause that satisfy a specified predicate. Consider the query "Find the names of all instructors in the Computer Science department who have salary greater than $70,000." This query can be written in SQL as:
```SQL
select name
from instructor
where dept_name = 'Comp. Sci.' and salary > 70000;
```
SQL allows the use of the logical connectives and, or, and not in the where clause. The operands of the logical connectives can be expressions involving the comparison operators <, <=, >, >=, =, and <>. SQL allows us to use the comparison operators to compare strings and arithmetic expressions, as well as special types, such as date types.

## Queries on Multiple Relations

