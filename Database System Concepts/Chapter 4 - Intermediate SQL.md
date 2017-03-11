# Join Expressions
## Join Conditions
In Section 3.3.3, we saw how to express natural joins, and we saw the join ... using clause, which is a form of natural join that only requires values to match on specified attributes. SQL supports another form of join, in which an arbitrary join condition can be specified.

***
 A NATURAL JOIN is a JOIN operation that creates an implicit join clause for you based on the common columns in the two tables being joined. Common columns are columns that have the same name in both tables.

A NATURAL JOIN can be an INNER join, a LEFT OUTER join, or a RIGHT OUTER join. The default is INNER join. 
***

The on condition allows a general predicate over the relations being joined. This predicate is written like a where clause predicate except for the use of the keyword on rather than where. Like the using condition, the on condition appears at the end of the join expression. Consider the following query, which has a join expression containing the on condition.
```SQL
select *
from student [INNER] join takes on student.ID = takes.ID;
```
The on condition above specifies that a tuple from student matches a tuple from takes if their ID values are equal. The join expression in this case is almost the same as the join expression student natural join takes, since the natural join operation also requires that for a student tuple and a takes tuple to match. The one difference is that the result has the ID attribute listed twice, in the join result, once for student and once for takes, even though their ID values must be the same.  

In fact, the above query is equivalent to the following query (in other words, they generate exactly the same results):
```SQL
select *
from student, takes
where student.ID = takes.ID ;
```

It may appear that the on condition is a redundant feature of SQL. However, there are two good reasons for introducing the on condition. First, we shall see shortly that for a kind of join called an outer join, on conditions do behave in a manner different from where conditions. Second, an SQL query is often more readable by humans if the join condition is specified in the on clause and the rest of the conditions appear in the where clause.

## Outer Joins

Suppose we wish to display a list of all students, displaying their ID, and name, dept_name, and tot_cred, along with the courses that they have taken. The following SQL query may appear to retrieve the required information:
```SQL
select *
from student natural join takes;
``` 
Unfortunately, the above query does not work quite as intended. Suppose that there is some student who takes no courses. Then the tuple in the student relation for that particular student would not satisfy the condition of a natural join with any tuple in the takes relation, and that student's data would not appear in the result. We would thus not see any information about students who have not taken any courses.

More generally, some tuples in either or both of the relations being joined may be "lost" in this way. The __outer join__ operation works in a manner similar to the join operations we have already studied, but preserve those tuples that would be lost in a join, by creating tuples in the result containing null values.  

There are in fact three forms of outer join:
* The __left [outer] join__ preserves tuples only in the relation named before (to the left of) the left outer join operation.
* The __right [outer] join__ preserves tuples only in the relation named after (to the right of) the right outer join operation.
* The __full [outer] join__ preserves tuples in both relations.

In contrast, the join operations we studied earlier that do not preserve nonmatched tuples are called __inner join__ operations, to distinguish them from the outer-join operations.  

We now explain exactly how each form of outer join operates. We can compute the left outer-join operation as follows. First, compute the result of the inner join as before. Then, for every tuple t in the left-hand-side relation that does not match any tuple in the right-hand-side relation in the inner join, add a tuple r to the result of the join constructed as follows:
* The attributes of tuple r that are derived from the left-hand-side relation are filled in with the values from tuple t.
* The remaining attributes of r are filled with null values.

We can write the query "Find all students who have not taken a course" as:
```SQL
select ID
from student natural left outer join takes
where course_id is null;
```
The __right outer join__ is symmetric to the left outer join. Tuples from the right-hand-side relation that do not match any tuple in the left-hand-side relation are padded with nulls and are added to the result of the right outer join.  

The __full outer join__ is a combination of the left and right outer-join types.  

The __on__ clause can be used with outer joins. The following query is identical to the first query we saw using "student natural left outer join takes," except that the attribute ID appears twice in the result.
```SQL
select *
from student left outer join takes on student.ID = takes.ID;
```
As we noted earlier, on and where behave differently for outer join. The reason for this is that outer join adds null-padded tuples only for those tuples that do not contribute to the result of the corresponding inner join. The on condition is part of the outer join specification, but a where clause is not.
```SQL
select * from instructor left outer join teaches on instructor.id = teaches.id and instructor.id = 101;
+-----+--------+----------------------+--------+------+-----------+----------+------+
| id  | name   | dept_name            | salary | id   | course_id | semester | year |
+-----+--------+----------------------+--------+------+-----------+----------+------+
|  99 | Jack   | Computer Engineering |   1150 | NULL |      NULL | NULL     | NULL |
| 100 | Andy   | Computer Engineering |   1600 | NULL |      NULL | NULL     | NULL |
| 101 | Wu     | E. Engineering       |   1800 |  101 |     10009 | Spring   | 2015 |
| 101 | Wu     | E. Engineering       |   1800 |  101 |     10010 | Summer   | 2015 |
| 101 | Wu     | E. Engineering       |   1800 |  101 |     10016 | Spring   | 2014 |
| 101 | Wu     | E. Engineering       |   1800 |  101 |     10016 | Spring   | 2015 |
| 102 | Brandt | E. Engineering       |   1800 | NULL |      NULL | NULL     | NULL |
| 103 | Katz   | Psychology           |   1000 | NULL |      NULL | NULL     | NULL |
| 104 | Gold   | Psychology           |   1100 | NULL |      NULL | NULL     | NULL |
| 105 | Edward | History              |   1000 | NULL |      NULL | NULL     | NULL |
| 106 | Lea    | Art                  |   1000 | NULL |      NULL | NULL     | NULL |
| 107 | Taylor | Biology              |   1600 | NULL |      NULL | NULL     | NULL |
+-----+--------+----------------------+--------+------+-----------+----------+------+

```
```SQL
select * from instructor left join teaches on instructor.id = teaches.id where  instructor.id = 101;
+-----+------+----------------+--------+------+-----------+----------+------+
| id  | name | dept_name      | salary | id   | course_id | semester | year |
+-----+------+----------------+--------+------+-----------+----------+------+
| 101 | Wu   | E. Engineering |   1800 |  101 |     10009 | Spring   | 2015 |
| 101 | Wu   | E. Engineering |   1800 |  101 |     10010 | Summer   | 2015 |
| 101 | Wu   | E. Engineering |   1800 |  101 |     10016 | Spring   | 2014 |
| 101 | Wu   | E. Engineering |   1800 |  101 |     10016 | Spring   | 2015 |
+-----+------+----------------+--------+------+-----------+----------+------+

```



# Views
SQL allows a "virtual relation" to be defined by a query, and the relation conceptually contains the result of the query. The virtual relation is not precomputed and stored, but instead is computed by executing the query whenever the virtual relation is used. Any such relation that is not part of the logical model, but is made visible to a user as a virtual relation, is called a __view__. It is possible to support a large number of views on top of any given set of actual relations.

## View Definition

```SQL
create view faculty as select ID,name,dept_name from instructor;
```

## Using Views in SQL Queries
View names may appear in a query any place where a relation name may appear.
```SQL
select course_id
from physics fall 2009
where building='Watson';
```

## Materialized Views

Certain database systems allow view relations to be stored, but they make sure that, if the actual relations used in the view definition change, the view is kept up-to-date. Such views are called materialized views.  

SQL does not define a standard way of specifying that a view is materialized, but many database systems provide their own SQL extensions for this task. Some database systems always keep materialized views up-to-date when the underlying relations change, while others permit them to become out of date, and periodically recompute them.

## Update of a View

Although views are a useful tool for queries, they present serious problems if we express updates, insertions, or deletions with them. The difficulty is that a modification to the database expressed in terms of a view must be translated to a modification to the actual relations in the logical model of the database.  

Suppose the view _faculty_, which we saw earlier, is made available to a clerk. Since we allow a view name to appear wherever a relation name is allowed, the clerk can write:
```SQL
insert into faculty
values ('30765', 'Green', 'Music');
```
This insertion must be represented by an insertion into the relation instructor, since instructor is the actual relation from which the database system constructs the view faculty. However, to insert a tuple into instructor, we must have some value for salary. There are two reasonable approaches to dealing with this insertion:
* Reject the insertion, and return an error message to the user.
* Insert a tuple ('30765', 'Green', 'Music', null) into the instructor relation.

Modifications are generally not permitted on view relations, except in limited cases. Different database systems specify different conditions under which they permit updates on view relations; see the database system manuals for details. The general problem of database modification through views has been the subject of substantial research, and the bibliographic notes provide pointers to some of this research.  
In general, an SQL view is said to be updatable (that is, inserts, updates or deletes can be applied on the view) if the following conditions are all satisfied by the query defining the view:
* The from clause has only one database relation.
* The select clause contains only attribute names of the relation, and does not have any expressions, aggregates, or distinct specification.
* Any attribute not listed in the select clause can be set to null; that is, it does not have a not null constraint and is not part of a primary key.
* The query does not have a group by or having clause.

# Integrity Constraints

Integrity constraints ensure that changes made to the database by authorized users do not result in a loss of data consistency. Thus, integrity constraints guard against accidental damage to the database.  

Examples of integrity constraints are:
* An instructor name cannot be null.
* No two instructors can have the same instructor ID .
* Every department name in the course relation must have a matching department name in the department relation.
* The budget of a department must be greater than $0.00.

Integrity constraints are usually identified as part of the database schema design process, and declared as part of the __create table__ command used to create relations. However, integrity constraints can also be added to an existing relation by using the command __alter table__ table-name __add__ constraint, where constraint can be any constraint on the relation. When such a command is executed, the system first ensures that the relation satisfies the specified constraint. If it does, the constraint is added to the relation; if not, the command is rejected.

## Not Null Constraint
The null value is a member of all domains, and as a result is a legal value for every attribute in SQL by default. For certain attributes, however, null values may be inappropriate. Consider a tuple in the student relation where name is null. Such a tuple gives student information for an unknown student; thus, it does not contain useful information. Similarly, we would not want the department budget to be null. In cases such as this, we wish to forbid null values, and we can do so by restricting the domain of the attributes name and budget to exclude null values, by declaring it as follows:
```SQL
name varchar(20) not null
budget numeric(12,2) not null
```
The not null specification prohibits the insertion of a null value for the attribute. Any database modification that would cause a null to be inserted in an attribute declared to be not null generates an error diagnostic.

## Unique Constraint

SQL also supports an integrity constraint:
```SSQL
unique (A1, A2, ..., Am)
```
The unique specification says that attributes A1, A2, ..., Am form a candidate key; that is, no two tuples in the relation can be equal on all the listed attributes. However, candidate key attributes are permitted to be null unless they have explicitly been declared to be not null.


## The check Clause
When applied to a relation declaration, the clause check(P) specifies a predicate P that must be satisfied by every tuple in a relation.  

A common use of the check clause is to ensure that attribute values satisfy specified conditions, in effect creating a powerful type system. For instance, a clause check (budget > 0) in the create table command for relation department would ensure that the value of budget is nonnegative.  

As another example, consider the following:
```SQL
create table section
(course_id varchar (8),
sec_id varchar (8),
semester varchar (6),
year numeric (4,0),
building varchar (15),
room_number varchar (7),
time_slot_id varchar (4),
primary key (course_id, sec_id, semester, year),
check (semester in (’Fall’, ’Winter’, ’Spring’, ’Summer’)));
```SQL
Here, we use the check clause to simulate an enumerated type, by specifying that semester must be one of 'Fall', 'Winter', 'Spring', or 'Summer'.

## Referential Integrity
Often, we wish to ensure that a value that appears in one relation for a given set of attributes also appears for a certain set of attributes in another relation. This condition is called referential integrity.

When a referential-integrity constraint is violated, the normal procedure is to reject the action that caused the violation (that is, the transaction performing the update action is rolled back). However, a foreign key clause can specify that if a delete or update action on the referenced relation violates the constraint, then, instead of rejecting the action, the system must take steps to change the tuple in the referencing relation to restore the constraint. Consider this definition of an integrity constraint on the relation course:
```SQL
create table course
( ...
foreign key (dept name) references department
on delete cascade
on update cascade,
. . . );
```
Because of the clause on delete cascade associated with the foreign-key declaration, if a delete of a tuple in department results in this referential-integrity constraint being violated, the system does not reject the delete. Instead, the delete "cascades" to the course relation, deleting the tuple that refers to the department that was deleted. Similarly, the system does not reject an update to a field referenced by the constraint if it violates the constraint; instead, the system updates the field dept_name in the referencing tuples in course to the new value as well. SQL also allows the foreign key clause to specify actions other than cascade, if the constraint is violated: The referencing field (here, dept_name) can be set to null (by using __set null__ in place of __cascade__), or to the default value for the domain (by using __set default__).

# SQL Data Types and Schemas (Incomplete)

# Authorization

## Granting and Revoking of Privileges
The SQL standard includes the privileges __select__, __insert__, __update__, and __delete__. The privilege __all privileges__ can be used as a short form for all the allowable privileges. A user who creates a new relation is given all privileges on that relation automatically.  

The SQL data-definition language includes commands to grant and revoke privileges. The grant statement is used to confer authorization. The basic form of this statement is:
```SQL
grant <privilege list>
on <relation name or view name>
to <user/role list>;
```





























