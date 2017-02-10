# Structure of Relational Databases

In the relational model the term __relation__ is used to refer to a table, while the term __tuple__ is used to refer to a row. Similarly, the term __attribute__ refers to a column of a table.  

For each attribute of a relation, there is a set of permitted values, called the domain of that attribute. Thus, the domain of the salary attribute of the instructor relation is the set of all possible salary values, while the domain of the name attribute is the set of all possible instructor names. We require that, for all relations r, the domains of all attributes of r be atomic. A domain is atomic if elements of the domain are considered to be indivisible units.

# Database Schema
When we talk about a database, we must differentiate between the database schema, which is the logical design of the database, and the database instance, which is a snapshot of the data in the database at a given instant in time.

The schema for department relation might be _department(dept name,_ _building,_ _budget)_
