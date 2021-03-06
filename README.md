# json_build extension

This PostgreSQL extension provides functions to help in building
JSON of arbitrary complexity.

The functions are:

* `build_json_object (VARIADIC "any")`
* `build_json_array (VARIADIC "any")`
* `json_object_agg  ("any", "any")`

All the functions return JSON. They can be called nested and combined, to build 
up complex tree structured JSON.

`VARIADIC  "any"` means that the functions will accept any number of arguments 
of any type, and Postgres will accept the call, although the functions
themselves do enforce certain rules. If called with no arguments an empty 
object or array is returned.

If an argument is an array, it is converted to a JSON array, if it is a record,
it is converted to a JSON object, if it is JSON it is passed through as is.

Object keys must be not null and scalar - use of arrays, records or JSON values
as keys is forbidden. 

`build_json_object` must get an even number of arguments - the odd numbered 
arguments (counting from 1) are the keys and the following even numbered 
arguments are the corresponding values.

`json_object_agg` aggregates any two columns into a json object.

Examples:

    SELECT build_json_object( 
           'a', build_json_object('b',false,'c',99), 
           'd', build_json_object('e',array[9,8,7]::int[],
               'f', (select row_to_json(r) from ( SELECT relkind, oid::regclass as name 
                                                  FROM pg_class WHERE relname = 'pg_class') r)));
                                            build_json_object                                        
    -------------------------------------------------------------------------------------------------
     {"a" : {"b" : false, "c" : 99}, "d" : {"e" : [9,8,7], "f" : {"relkind":"r","name":"pg_class"}}}
    (1 row)

    SELECT build_json_array('a',1,'b',1.2,'c',true,'d',null,'e',json '{"x": 3, "y": [1,2,3]}');
                               build_json_array                            
    -----------------------------------------------------------------------
     ["a", 1, "b", 1.2, "c", true, "d", null, "e", {"x": 3, "y": [1,2,3]}]
    (1 row)

    SELECT build_json_object('stuff',json_object_agg(k,v)) 
    FROM (values ('k1','v1'),('k2','v2')) AS x(k,v);
                build_json_object             
    ------------------------------------------
     {"stuff" : { "k1" : "v1", "k2" : "v2" }}



