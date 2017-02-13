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
