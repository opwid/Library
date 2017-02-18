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

An an example, suppose we want to answer the query "Retrieve the names of all instructors, along with their department names and department building name."  

Looking at the schema of the relation instructor, we realize that we can get the department name from the attribute dept_name, but the department building name is present in the attribute building of the relation department. To answer the query, each tuple in the instructor relation must be matched with the tuple in the department relation whose dept_name value matches the dept_name value of the instructor tuple.  

In SQL, to answer the above query, we list the relations that need to be accessed in the from clause, and specify the matching condition in the where clause. The above query can be written in SQL as
```SQL
select name, instructor.dept_name, building
from instructor, department
where instructor.dept_name= department.dept_name;
```
Note that the attribute dept_name occurs in both the relations instructor and department, and the relation name is used as a prefix (in instructor.dept_name, and department.dept_name) to make clear to which attribute we are referring. In contrast, the attributes name and building appear in only one of the relations, and therefore do not need to be prefixed by the relation name.  

The role of each clause is as follows:
* The select clause is used to list the attributes desired in the result of a query.
* The from clause is a list of the relations to be accessed in the evaluation of the query. The from clause by itself defines a __Cartesian product__ of the relations listed in the clause.
* The where clause is a predicate involving attributes of the relation in the from clause.

The predicate in the where clause is used to restrict the combinations created by the Cartesian product to those that are meaningful for the desired answer. We would expect a query involving instructor and teaches to combine a particular tuple t in instructor with only those tuples in teaches that refer to the same instructor to which t refers. That is, we wish only to match teaches tuples with instructor tuples that have the same ID value. The following SQL query ensures this condition, and outputs the instructor name and course identifiers from such matching tuples.
```SQL
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID;
```

## The Natural Join 

In our example query that combined information from the instructor and teaches table, the matching condition required instructor.ID to be equal to teaches.ID. These are the only attributes in the two relations that have the same name. In fact this is a common case; that is, the matching condition in the from clause most often requires all attributes with matching names to be equated. To make the life of an SQL programmer easier for this common case, SQL supports an operation called the __natural join__, which we describe below. In fact SQL supports several other ways in which information from two or more relations can be joined together. We have already seen how a Cartesian product along with a where clause predicate can be used to join information from multiple relations.  

The natural join operation operates on two relations and produces a relation as the result. Unlike the Cartesian product of two relations, which concatenates each tuple of the first relation with every tuple of the second, natural join considers only those pairs of tuples with the same value on those attributes that appear in the schemas of both relations. So, going back to the example of the relations instructor and teaches, computing instructor natural join teaches considers only those pairs of tuples where both the tuple from instructor and the tuple from teaches have the same value on the common attribute, ID.  

Consider the query "For all instructors in the university who have taught some course, find their names and the course ID of all courses they taught", which we wrote earlier as:
```SQL
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID ;
```
This query can be written more concisely using the natural-join operation in SQL as:
```SQL
select name, course_id
from instructor natural join teaches;
```
Both of the above queries generate the same result.  

A from clause in an SQL query can have multiple relations combined using natural join, as shown here:  

select A<sub>1</sub> , A<sub>2</sub>, ..., A<sub>n</sub>  
from r<sub>1</sub> natural join r<sub>2</sub> natural join ... natural join r<sub>m</sub>  
where P;  

For example, suppose we wish to answer the query "List the names of instructors along with the the titles of courses that they teach." The query can be written in SQL as follows:
```SQL
select name, title
from instructor natural join teaches, course
where teaches.course_id = course.course_id;
```
The natural join of instructor and teaches is first computed, as we saw earlier, and a Cartesian product of this result with course is computed, from which the where clause extracts only those tuples where the course identifier from the join result matches the course identifier from the course relation. Note that teaches.course_id in the where clause refers to the course_id field of the natural join result, since this field in turn came from the teaches relation.  

To provide the benefit of natural join while avoiding the danger of equating attributes erroneously, SQL provides a form of the natural join construct that allows you to specify exactly which columns should be equated. This feature is illustrated by the following query:
```SQL
select name, title
from (instructor natural join teaches) join course using (course_id);
```
The operation __join ... using__ requires a list of attribute names to be specified. Both inputs must have attributes with the specified names. Consider the operation r<sub>1</sub> join r<sub>2</sub> using(A<sub>1</sub>, A<sub>2</sub>). The operation is similar to r<sub>1</sub> natural join r<sub>2</sub>, except that a pair of tuples t<sub>1</sub> from r<sub>1</sub> and t<sub>2</sub> from r<sub>2</sub> match if t<sub>1</sub>.A<sub>1</sub> = t<sub>2</sub>.A<sub>1</sub> and t<sub>1</sub>.A<sub>2</sub> = t<sub>2</sub>.A<sub>2</sub>; even if r<sub>1</sub> and r<sub>2</sub> both have an attribute named A<sub>3</sub>, it is not required that t<sub>1</sub>.A<sub>3</sub> = t<sub>2</sub>.A<sub>3</sub>.

# Additional Basic Operations
## The Rename Operation
Consider again the query that we used earlier:
```SQL
select name, course_id
from instructor, teaches
where instructor.ID = teaches.ID;
```
The names of the attributes in the result are derived from the names of the attributes in the relations in the from clause.  

We cannot, however, always derive names in this way, for several reasons: First, two relations in the from clause may have attributes with the same name, in which case an attribute name is duplicated in the result. Second, if we used an arithmetic expression in the select clause, the resultant attribute does not have a name. Third, even if an attribute name can be derived from the base relations as in the preceding example, we may want to change the attribute name in the result `instructor.name`. Hence, SQL provides a way of renaming the attributes of a result relation. It uses the as clause, taking the form:
```SQL
select name as instructor_name, course_id
from instructor, teaches
where instructor.ID = teaches.ID;
```
```SQL
select T.name, S.course_id
from instructor as T, teaches as S
where T.ID = S.ID;
```
Another reason to rename a relation is a case where we wish to compare tuples in the same relation.
```SQL
select distinct T.name
from instructor as T, instructor as S
where T.salary > S.salary and S.dept name = 'Biology';
```
Observe that we could not use the notation instructor.salary, since it would not be clear which reference to instructor is intended.

## String Operations
SQL specifies strings by enclosing them in single quotes, for example, 'Computer'. A single quote character that is part of a string can be specified by using two single quote characters; for example, the string 'It's right' can be specified by 'It''s right'. Another way is using single quote character inside double quotes; for examle, "It's right".  

SQL also permits a variety of functions on character strings, such as concatenating (using "||"), extracting substrings, finding the length of strings, converting strings to uppercase (using the function upper(s) where s is a string) and lowercase (using the function lower(s)), removing spaces at the end of the string (using trim(s)) and so on.
```SQL
select concat("Departmant name:", DEPT_NAME) from department;
+---------------------------------------+
| concat("Departmant name:", DEPT_NAME) |
+---------------------------------------+
| Departmant name:CE                    |
| Departmant name:EE                    |
| Departmant name:History               |
| Departmant name:Arts                  |
+---------------------------------------+

select concat("Departmant name:", DEPT_NAME) as department_name from department;
+---------------------------------------+
| department_name                       |
+---------------------------------------+
| Departmant name:CE                    |
| Departmant name:EE                    |
| Departmant name:History               |
| Departmant name:Arts                  |
+---------------------------------------+

```
Pattern matching can be performed on strings, using the operator __like__. We describe patterns by using two special characters:

* Percent (%): The % character matches any substring.
* Underscore (_): The character matches any character.

Consider the query "Find the names of all departments whose building name includes the substring 'Watson'." This query can be written as:
```SQL
select dept_name
from department
where building like '%Watson%';
```

SQL allows us to search for mismatches instead of matches by using the __not like__ comparison operator.

## Ordering the Display of Tuples

SQL offers the user some control over the order in which tuples in a relation are displayed. The __order by__ clause causes the tuples in the result of a query to appear in sorted order.
```SQL
select name
from instructor
where dept_name = ’Physics’
order by name;
```
By default, the order by clause lists items in ascending order. To specify the sort order, we may specify desc for descending order or asc for ascending order. Furthermore, ordering can be performed on multiple attributes. Suppose that we wish to list the entire instructor relation in descending order of salary. If several instructors have the same salary, we order them in ascending order by name. We express this query in SQL as follows:
```SQL
select *
from instructor
order by salary desc, name asc;
```
## Where Clause Predicates

SQL includes a __between__ comparison operator to simplify where clauses that specify that a value be less than or equal to some value and greater than or equal to some other value. If we wish to find the names of instructors with salary amounts between $90,000 and $100,000, we can use the between comparison to write:
```SQL
select name
from instructor
where salary between 90000 and 100000;
```
instead of:
```SQL
select name
from instructor
where salary <= 100000 and salary >= 90000;
```
Similarly, we can use the __not between__ comparison operator.

# Set Operations
## The Union Operation

To find the set of all courses taught either in Fall 2009 or in Spring 2010, or both, we write:
```SQL
(select course_id
from section
where semester = 'Fall' and year= 2009)
union
(select course_id
from section
where semester = 'Spring' and year= 2010);
```
The union operation automatically eliminates duplicates, unlike the select clause. If we want to retain all duplicates, we must write union all in place of union.

## The Intersect Operation

To find the set of all courses taught in the Fall 2009 as well as in Spring 2010 we write:
```SQL
(select course_id
from section
where semester = 'Fall' and year= 2009)
intersect
(select course_id
from section
where semester = 'Spring' and year= 2010);
```
The intersect operation automatically eliminates duplicates. If we want to retain all duplicates, we must write intersect all in place of intersect.

## The Except Operation
To find all courses taught in the Fall 2009 semester but not in the Spring 2010 semester, we write:
```SQL
(select course_id
from section
where semester = 'Fall' and year= 2009)
except
(select course_id
from section
where semester = 'Spring' and year= 2010);
```
If we want to retain duplicates, we must write except all in place of except. 

# Null Values

Comparisons involving nulls are more of a problem. For example, consider the comparison "1 < null". It would be wrong to say this is true since we do not know what the null value represents. But it would likewise be wrong to claim this expression is false; if we did, "not (1 < null)" would evaluate to true, which does not make sense. SQL therefore treats as unknown the result of any comparison involving a null value (other than predicates is null and is not null, which are described later in this section). This creates a third logical value in addition to true and false.  

Since the predicate in a where clause can involve Boolean operations such as and, or, and not on the results of comparisons, the definitions of the Boolean operations are extended to deal with the value unknown.

* and: The result of true and unknown is unknown, false and unknown is false, while unknown and unknown is unknown.
* or: The result of true or unknown is true, false or unknown is unknown, while unknown or unknown is unknown.
* not: The result of not unknown is unknown.


SQL uses the special keyword null in a predicate to test for a null value. Thus, to find all instructors who appear in the instructor relation with null values for salary, we write:
```SQL
select name
from instructor
where salary is null;
```
The predicate is not null succeeds if the value on which it is applied is not null.

# Aggregate Functions

Aggregate functions are functions that take a collection (a set or multiset) of values as input and return a single value. SQL offers five built-in aggregate functions:

* Average: avg
* Minimum: min
* Maximum: max
* Total: sum
* Count: count

The input to sum and avg must be a collection of numbers, but the other operators can operate on collections of nonnumeric data types, such as strings, as well.

## Basic Aggregation
Consider the query "Find the average salary of instructors in the Computer Science department." We write this query as follows:
```SQL
select avg (salary)
from instructor
where dept_name= ’Comp. Sci.’;
```


There are cases where we must eliminate duplicates before computing an aggregate function. If we do want to eliminate duplicates, we use the keyword __distinct__ in the aggregate expression. An example arises in the query "Find the total number of instructors who teach a course in the Spring 2010 semester." In this case, an instructor counts only once, regardless of the number of course sections that the instructor teaches. The required information is contained in the relation teaches, and we write this query as follows:
```SQL
select count(distinct ID )
from teaches
where semester = 'Spring' and year = 2010;
```
Because of the keyword distinct preceding ID, even if an instructor teaches more than one course, she is counted only once in the result.

## Aggregation with Grouping
There are circumstances where we would like to apply the aggregate function not only to a single set of tuples, but also to a group of sets of tuples; we specify this wish in SQL using the group by clause. The attribute or attributes given in the group by clause are used to form groups. Tuples with the same value on all attributes in the group by clause are placed in one group. As an illustration, consider the query "Find the average salary in each department." We write this query as follows:
```SQL
select dept_name, avg (salary) as avg salary
from instructor
group by dept_name;
```
In contrast, consider the query "Find the average salary of all instructors." We write this query as follows:
```SQL
select avg (salary)
from instructor;
```
In this case the group by clause has been omitted, so the entire relation is treated as a single group.  

At times, it is useful to state a condition that applies to groups rather than to tuples. For example, we might be interested in only those departments where the average salary of the instructors is more than $42,000. This condition does not apply to a single tuple; rather, it applies to each group constructed by the group by clause. To express such a query, we use the having clause of SQL. SQL applies predicates in the having clause after groups have been formed, so aggregate functions may be used. We express this query in SQL as follows:
```SQL

select dept_name, avg (salary) as avg_salary
from instructor
group by dept_name
having avg (salary) > 42000;
```
To illustrate the use of both a having clause and a where clause in the same query, we consider the query "For each course section offered in 2009, find the average total credits (tot_cred) of all students enrolled in the section, if the section had at least 2 students."
```SQL
select course id, semester, year, sec_id, avg (tot_cred)
from takes natural join student
where year = 2009
group by course id, semester, year, sec_id
having count(ID) >= 2;
```
## Aggregation with Null and Boolean Values

In general, aggregate functions treat nulls according to the following rule: All aggregate functions except count (\*) ignore null values in their input collection. As a result of null values being ignored, the collection of values may be empty. The count of an empty collection is defined to be 0, and all other aggregate operations return a value of null when applied on an empty collection.

# Nested Subqueries
## Set Membetship

SQL allows testing tuples for membership in a relation. The __in__ connective tests for set membership, where the set is a collection of values produced by a select clause. The __not in__ connective tests for the absence of set membership.  

As an illustration, reconsider the query "Find all the courses taught in the both the Fall 2009 and Spring 2010 semesters."
We begin by finding all courses taught in Spring 2010, and we write the subquery
```SQL
(select course_id
from section
where semester = 'Spring' and year= 2010)
```
We then need to find those courses that were taught in the Fall 2009 and that appear in the set of courses obtained in the subquery. We do so by nesting the subquery in the where clause of an outer query. The resulting query is
```SQL
select distinct course_id
from section
where semester = 'Fall' and year= 2009 and
course_id in (select course_id
from section
where semester = 'Spring' and year= 2010);
```

We use the not in construct in a way similar to the in construct.

The in and not in operators can also be used on enumerated sets. The following query selects the names of instructors whose names are neither "Mozart" nor "Einstein".
```SQL
select distinct name
from instructor
where name not in ('Mozart', 'Einstein');
```
## Set Comparison
As an example of the ability of a nested subquery to compare sets, consider the query "Find the names of all instructors whose salary is greater than at least one instructor in the Biology department." In Section 3.4.1, we wrote this query as follows:
```SQL
select distinct T.name
from instructor as T, instructor as S
where T.salary > S.salary and S.dept_name = 'Biology';
```
SQL does, however, offer an alternative style for writing the preceding query. The phrase "greater than at least one" is represented in SQL by __> some__.

```SQL
select name
from instructor
where salary > some 
(select salary
from instructor
where dept name = 'Biology');
```

SQL also allows < some, <= some, >= some, = some, and <> some comparisons.  

As another example of set comparisons, consider the query "Find the departments that have the highest average salary." We begin by writing a query to find all average salaries, and then nest it as a subquery of a larger query that finds those departments for which the average salary is greater than or equal to all average salaries. The construct __> all__ corresponds to the phrase "greater than all."
```SQL
select dept_name
from instructor
group by dept_name
having avg (salary) >= all 
(select avg (salary)
from instructor
group by dept_name);
```

## Test for Empty Relations

SQL includes a feature for testing whether a subquery has any tuples in its result. The exists construct returns the value true if the argument subquery is nonempty. Using the __exists__ construct, we can write the query "Find all courses taught in both the Fall 2009 semester and in the Spring 2010 semester" in still another way:
```SQL
select course_id
from section as S
where semester = 'Fall' and year= 2009 and
exists (select *
from section as T
where semester = 'Spring' and year= 2010 and
S.course id= T.course id);
```
The above query also illustrates a feature of SQL where a correlation name from an outer query (S in the above query), can be used in a subquery in the where clause. A subquery that uses a correlation name from an outer query is called a correlated subquery.  


We can test for the nonexistence of tuples in a subquery by using the __not exists__ construct.
## Test for the Absence of Duplicate Tuples
SQL includes a boolean function for testing whether a subquery has duplicate tuples in its result. The unique construct returns the value true if the argument subquery contains no duplicate tuples. Using the unique construct, we can write the query "Find all courses that were offered at most once in 2009" as follows:
```SQL
select T.course_id
from course as T
where unique (select R.course_id
from section as R
where T.course id= R.course_id and
R.year = 2009);
```

We can test for the existence of duplicate tuples in a subquery by using the __not unique__ construct.

## Subqueries in the From Clause

Consider the query "Find the average instructors' salaries of those departments where the average salary is greater than $42,000." We wrote this query in by using the having clause. We can now rewrite this query, without using the having clause, by using a subquery in the from clause, as follows:
```SQL
select dept_name, avg salary
from (select dept_name, avg (salary) as avg_salary
from instructor
group by dept_name)
where avg_salary > 42000;
```
## The with Clause
The with clause provides a way of defining a temporary relation whose definition is available only to the query in which the with clause occurs. Consider the following query, which finds those departments with the maximum budget.
```SQL
with max_budget(value) as
(select max(budget)
from department)
select budget
from department, max_budget
where department.budget = max_budget.value;
```
The with clause defines the temporary relation max budget, which is used in the immediately following query.  

For example, suppose we want to find all departments where the total salary is greater than the average of the total salary at all departments. We can write the query using the with clause as follows.
```SQL
with dept_total(dept_name, value) as
(select dept_name, sum(salary)
from instructor
group by dept_name),
dept_total avg(value) as
(select avg(value)
from dept_total)
select dept_name
from dept_total, dept_total_avg
where dept total.value >= dept_total_avg.value;
```


# Modification of Database

## Deletion
We can delete only whole tuples; we cannot delete values on only particular attributes. SQL expresses a deletion by
```SQL
delete from r
where P;
```
The where clause can be omitted, in which case all tuples in r are deleted. Note that a delete command operates on only one relation. If we want to delete tuples from several relations, we must use one delete command for each relation. The predicate in the where clause may be as complex as a select command's where clause.

## Insertion
The simplest insert statement is a request to insert one tuple.
```SQL
insert into course
values ('CS-437', 'Database Systems', 'Comp. Sci.', 4);
```
```SQL
insert into course (title, course id, credits, dept name)
values ('Database Systems', 'CS-437', 4, 'Comp. Sci.');
```

More generally, we might want to insert tuples on the basis of the result of a query. Suppose that we want to make each student in the Music department who has earned more than 144 credit hours, an instructor in the Music department, with a salary of $18,000. We write:
```SQL
insert into instructor
select ID, name, dept_name, 18000
from student
where dept_name = ’Music’ and tot_cred > 144;
```
## Updates

Suppose that annual salary increases are being made, and salaries of all instructors are to be increased by 5 percent. We write:
```SQL
update instructor
set salary= salary * 1.05;
```
If a salary increase is to be paid only to instructors with salary of less than $70,000, we can write:
```SQL
update instructor
set salary = salary * 1.05
where salary < 70000;
```
SQL provides a __case__ construct that we can use to perform both the updates with a single update statement, avoiding the problem with the order of updates.
```SQL
update instructor
set salary = 
case
when salary <= 100000 then salary * 1.05
else salary * 1.03
end
```







