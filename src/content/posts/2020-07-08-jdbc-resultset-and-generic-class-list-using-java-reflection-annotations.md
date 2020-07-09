---
template: blog-post
title: JDBC ResultSet and Generic Class List Using Java Reflection Annotations
slug: java-jdbc
date: 2020-07-08 14:08
description: java jdbc
---
When working with web projects, the frequency required to query the database and get results is quite high. This involves a lot of code in your DAO classes and will affect readability and connection backlogs.

If we observe closely, its code is static, except for the `Model`class type and query in it.

For example, if we need Employee details, we will create the code with`Connection`,`Statement`,`ResultSet`, and closing blocks. If we require`Department`details, we need to do the same as before, except we change the`List`type to`Department`and the query we run through it.

Based on this experience, I came across one article by [Roberto Benitez](http://baseprogramming.com/blog1/author/administrator/)on this [blog](http://baseprogramming.com/blog1/2017/08/24/automating-jdbc-crud-operations-with-reflection/).

Please go through the above article and come back here to see how I used it to serve the previously discussed problems.

To implement, check out the steps below:

1.  Create a custom annotation
2.  Create a `Model` class, which contains mapping fields to the `ResultSet` column names with the created annotation.
3.  Call the `ResultSet`
4.  Load the `ResultSet` for each value into the object
5.  Check for the `Primitive` type
6.  Auto-box the `Primitive` type class.

### **Create the of Custom Annotation**

Here is how we created the custom annotation:VIM

     public Class Employee{
    		@DBTable(columnName ="emp_id")
    		private int empId;
    		@DBTable(columnName ="emp_name")
    		private String empName;
    		//Getters and setters
    		//Default constructor // mandatory
    		}
### **Creation of the Model Class**

Here is how we created the  `Model`  class, which contains mapping fields for the  `ResultSet`  column names with the created annotation:

       public Class Employee{
    @DBTable(columnName ="emp_id")
    private int empId;
    
    @DBTable(columnName ="emp_name")
    private String empName;
    
    //Getters and setters
    //Default constructor // mandatory
    }
### **Calling the ResultSet**

`String query =”select emp_id,emp_name from employee”`  
In the above query, column names should be equal to the  `@DBTable`  annotation column names.

    public <T> List<T> selectQuery(Class<T> type, query) throws SQLException {
        List<T> list = new ArrayList<T>();
        try (Connection conn = dataSource.getConnection()) {
            try (Statement stmt = conn.createStatement()) {
                try (ResultSet rst = stmt.executeQuery(query)) {
                    while (rst.next()) {
                        T t = type.newInstance();
                        loadResultSetIntoObject(rst, t);// Point 4
                        list.add(t);
                    }
                }
            } catch (InstantiationException | IllegalAccessException e) {
                throw new RuntimeException("Unable to get the records: " + e.getMessage(), e);
            }
        }
        return list;
### Loading the ResultSet for Each Value Into the Object
This looks quite interesting. Here's how Java annotations help us:

    public static void loadResultSetIntoObject(ResultSet rst, Object object)
            throws IllegalArgumentException, IllegalAccessException, SQLException {
        Class<?> zclass = object.getClass();
        for (Field field : zclass.getDeclaredFields()) {
            field.setAccessible(true);
            DBTable column = field.getAnnotation(DBTable.class);
            Object value = rst.getObject(column.columnName());
           Class<?> type = field.getType();
            if (isPrimitive(type)) {//check primitive type(Point 5)
                Class<?> boxed = boxPrimitiveClass(type);//box if primitive(Point 6)
                value = boxed.cast(value);
            }
            field.set(object, value);
        }
    }
### Check for Primitive Types
This will return a  `Primitive`  type:

    public static boolean isPrimitive(Class<?> type) {
        return (type == int.class || type == long.class || type == double.class || type == float.class
                || type == boolean.class || type == byte.class || type == char.class || type == short.class);
    }
### Auto-Boxing to Primitive Type Class
Here is the code for how we implemented this:
public static Class<?> boxPrimitiveClass(Class<?> type) {
   

     if (type == int.class) {
            return Integer.class;
        } else if (type == long.class) {
            return Long.class;
        } else if (type == double.class) {
            return Double.class;
        } else if (type == float.class) {
            return Float.class;
        } else if (type == boolean.class) {
            return Boolean.class;
        } else if (type == byte.class) {
            return Byte.class;
        } else if (type == char.class) {
            return Character.class;
        } else if (type == short.class) {
            return Short.class;
        } else {
            String string = "class '" + type.getName() + "' is not a primitive";
            throw new IllegalArgumentException(string);
        }
    }
We have made all arrangements to call our magic method with the help of [Roberto Benitez](http://baseprogramming.com/blog1/author/administrator/). Now, we will try to call this method and see the magic behind it all.
## Use Cases
By using`selectQuery`, this annotation can provide whatever class name you want from the list. You just need to provide the query that needs to run against the DB.
**Use case 1**: I want all the details of the student: `selectQuery(Student.class, "SELECT * FROM STUDENT")`
**Use case 2**: I want all of the details of the Department: `selectQuery(Department.class, "SELECT * FROM DEPARTMENT")`
**Use case 3**: I want all of the students in the Department and their enrolled courses. Quite a bit overhead, right? Dont worry too much. The solution is as simple as those mentioned above. Just make the `Model`  class with as many fields as you want, but make sure your query (bit joins) return all fields in the `Model` class.
**Note**: Please make sure the `@columnName` annotation value name of the fields for your Model class and column names of the DB result are the same. Keep whatever name you want for the fields.
**PS**: 

> I am not concerned about the performance, as of now, because I use
> this as a `Helper` method, which I want to use frequently with less
> code.
