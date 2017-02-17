# Structure of Relational Databases

In the relational model the term __relation__ is used to refer to a table, while the term __tuple__ is used to refer to a row. Similarly, the term __attribute__ refers to a column of a table.  

For each attribute of a relation, there is a set of permitted values, called the domain of that attribute. Thus, the domain of the salary attribute of the instructor relation is the set of all possible salary values, while the domain of the name attribute is the set of all possible instructor names. We require that, for all relations r, the domains of all attributes of r be atomic. A domain is atomic if elements of the domain are considered to be indivisible units.

# Database Schema
When we talk about a database, we must differentiate between the database schema, which is the logical design of the database, and the database instance, which is a snapshot of the data in the database at a given instant in time.

The schema for department relation might be _department(dept_name,_ _building,_ _budget)_

# Keys
We must have a way to specify how tuples within a given relation are distinguished. This is expressed in terms of their attributes. That is, the values of the attribute values of a tuple must be such that they can uniquely identify the tuple. In other words, no two tuples in a relation are allowed to have exactly the same value for all attributes.

A __superkey__ is a set of one or more attributes that, taken collectively, allow us to identify uniquely a tuple in the relation. For example, the _ID_ attribute of the relation instructor is sufficient to distinguish one instructor tuple from another. Thus, _ID_ is a superkey. The _name_ attribute of instructor, on the other hand, is not a superkey, because several instructors might have the same name.  

A superkey may contain extraneous attributes. For example, the combination of _ID_ and _name_ is a superkey for the relation instructor. If K is a superkey, then so is any superset of K. We are often interested in superkeys for which no proper subset is a superkey. Such minimal superkeys are called __candidate keys__.  

It is possible that several distinct sets of attributes could serve as a candidate key. Suppose that a combination of name and dept_name is sufficient to distinguish among members of the instructor relation. Then, both {_ID_} and {_name_, _dept_name_} are candidate keys. Although the attributes _ID_ and _name_ together can distinguish instructor tuples, their combination, {_ID_, _name_}, does not form a candidate key, since the attribute _ID_ alone is a candidate key.  

We shall use the term __primary key__ to denote a candidate key that is chosen by the database designer as the principal means of identifying tuples within a relation. A key (whether primary, candidate, or super) is a property of the entire relation, rather than of the individual tuples.  

A relation, say r<sub>1</sub>, may include among its attributes the primary key of another relation, say r<sub>2</sub>. This attribute is called a __foreign key__ from r<sub>1</sub>, referencing r<sub>2</sub>. The relation r<sub>1</sub> is also called the __referencing relation__ of the foreign key dependency, and r<sub>2</sub> is called the __referenced relation__ of the foreign key. For example, the attribute _dept_name_ in instructor is a foreign key from _instructor_, referencing _department_, since _dept_name_ is the primary key of department.

# Schema Diagrams
A database schema, along with primary key and foreign key dependencies, can be depicted by schema diagrams. Figure 2.8 shows the schema diagram for our university organization. Each relation appears as a box, with the relation name at the top in blue, and the attributes listed inside the box. Primary key attributes are shown underlined. Foreign key dependencies appear as arrows from the foreign key attributes of the referencing relation to the primary key of the referenced relation.  

![Figure 2.8](https://github.com/opwid/Library/blob/master/Database%20System%20Concepts/Images/Figure%202.8.png)

# Relational Operations  

![Relational Operations](https://github.com/opwid/Library/blob/master/Database%20System%20Concepts/Images/Relational%20Operations.png)

