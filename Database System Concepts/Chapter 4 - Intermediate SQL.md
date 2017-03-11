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
* The __left outer join__ preserves tuples only in the relation named before (to the left of) the left outer join operation.
* The __right outer join__ preserves tuples only in the relation named after (to the right of) the right outer join operation.
* The __full outer join__ preserves tuples in both relations.

In contrast, the join operations we studied earlier that do not preserve nonmatched tuples are called __inner join__ operations, to distinguish them from the outer-join operations.  

We now explain exactly how each form of outer join operates. We can compute the left outer-join operation as follows. First, compute the result of the inner join as before. Then, for every tuple t in the left-hand-side relation that does not match any tuple in the right-hand-side relation in the inner join, add a tuple r to the result of the join constructed as follows:
* The attributes of tuple r that are derived from the left-hand-side relation are filled in with the values from tuple t.
* The remaining attributes of r are filled with null values.

We can write the query "Find all students who have not taken a course" as:
```SQL
select ID
from student natural left outer join takes
where course id is null;
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

















