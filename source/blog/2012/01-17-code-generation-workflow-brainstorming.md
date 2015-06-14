@{
    Layout = "post";
    Title = "Code generation workflow brainstorming";
    Date = "2012-01-17T18:00:00";
    Tags = "mysql codegen database";
    Description = "Code generation workflow brainstorming";
}

Since starting work at [Pebble {code}](http://pebblecode.com/) one of the exciting new areas that I’ve been looking into is [code generation (or automatic programming)](http://en.wikipedia.org/wiki/Automatic_programming). The use case is simply auto-generating the bulk of database access code directly from the database model its self allowing quicker changes to the core model as you’re not having to re-write database access code along with every model change. This also allows for a higher level of abstraction, therefore allowing higher productivity with less code to directly maintain.

The current code generation approach we’re using is based around the open source [MyGeneration](http://www.mygenerationsoftware.com/) tool. For the design of the model we’re using [MySQL workbench](http://www.mysql.com/products/workbench/) for modelling the database. For the database interaction we’re using the [Apache iBatis project](http://code.google.com/p/mybatisnet/) for database mapping.

The current workflow for making a change to the model goes something like this:

1. Create table(s) in the MySql workbench model
2. Export updated create script
  1. Also create any migration scripts required to update the existing database with your changes
3. Re-create your local development database with the changes
4. Open MyGeneration and add any required metadata
5. Select the tables and objects you want to re-generate
6. Execute code generation
7. Perform required manual integration of the generated code
8. Add any additional handmade code to compliment the generated code

This current workflow can introduce friction to integration in two main areas:
1. Generating code off the local database does not ensure that the database has the latest set of changes, or is even generated from the current branch of development. Re-deployment of the database can also be a time-consuming process if it’s having to be run multiple times.
2. The MyGeneration tool has a single install path and is therefore a shared component, not specific to the current branch of development or included alongside the source control.

Therefore, my current ideas around this area are to research the feasibility of two main changes.

## Remove deployment to an actual database before generating code
The ideal solution here would be to have a method of extracting the database metadata from as close to the original model as possible. A good point of interception could be the SQL database create script. Theoretically it could be possible to work directly with the workbench files, however these are far less stable in standardisation than SQL and would be more likely to change and require more on-going work, whereas the SQL syntax is very strictly defined.

My current research in this area is to use [ANTLR](http://www.antlr.org/) to parse the [MySQL grammar](http://www.antlr.org/grammar/1242202695031/mysql-grammar.zip) (zip file), and create an object model to represent the created database. Initially this would just be supporting table creation statements, but could theoretically also extend to views and stored procedures. A good example of the object model is the [interfaces](http://mygeneration.svn.sourceforge.net/viewvc/mygeneration/trunk/mymeta/Interfaces/) used in the [MyMeta project](http://mygeneration.svn.sourceforge.net/viewvc/mygeneration/trunk/mymeta/) of MyGeneration.

## Integrate code generation into the IDE
I’m looking specifically at the [T4 Text Templates](http://msdn.microsoft.com/en-us/library/bb126445.aspx). This tool is specifically quite useful as it has tight integration with Visual Studio (and other .Net IDEs) as well as being able to be run on the [command line directly](http://msdn.microsoft.com/en-us/library/bb126245.aspx).

The T4 engine is designed specifically to create a single output file off of a single template but can also be adapted to [generate multiples files](http://www.olegsych.com/2008/03/how-to-generate-multiple-outputs-from-single-t4-template/). Using the MyGeneration tool to generate multiple files also uses a similar technique. This would therefore mean that there is a single control template which gets the input model using the previous technique, then runs all the sub-templates to create the files in the correct locations.
