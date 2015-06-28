@{
    Layout = "post";
    Title = "Reflections of a .Net'er in a Node World (Some F# Language Ideas)";
    Date = "2014-08-20T21:46:00";
    Tags = "fsharp node js npm";
    Description = "This article is a short list of pie-in-the-sky kind of ideas, which I guess are probably complete unfeasible and fraught with difficult edge cases, but are my way of asking the question of are we (the F# community) pushing for the very best language features, or are there any areas in which we settle for second-class solutions?";
}

I recently got the chance to have a brief change of scenery and join a project developing in node.js. Though I am very happy to get back to my comfort zone with .Net and F#, the experience gave me exposure to some nice new ideas, and got me thinking about what might be possible with F#, given that F# is also an open language with a great culture of trying new things and embracing change.

This article is a short list of pie-in-the-sky kind of ideas, which I guess are probably complete unfeasible and fraught with difficult edge cases, but are my way of asking the question of are we (the F# community) pushing for the very best language features, or are there any areas in which we settle for second-class solutions?

## Project Files
A challenge of being a .Net developer in a Javascript world is to want to utilise the Visual Studio tool chain while still playing nicely with other people making changes. One issue in this area is that Visual Studio requires the use of project files explicitly naming each file being used.

While the MSBuild in the underlying project files does support wildcards for file inclusion, Visual Studio really doesn't work well with wildcards when adding, removing or trying to organise files.

So, I guess this question is directly to Visual Studio rather than F#: How difficult would it be to make project files lighter-weight and work better with wildcards?

## File Dependencies
One of the only good reasons for the explicit listing of files in project files is to specify the order of F# files because of having to be passed to the compiler in order based on the dependencies between each of the files.

The trend in the Javascript ecosystem is to move towards the likes of Browserify or node's modules for dealing with file linking where you simply 'reference' the name of the file that you depend on such as:

    [lang=js]
    var foo = require("foo.js");

Then, rather than having a 'project file' listing all the files the need to be built, you just tell the compiler about the root file, and it will dynamically include the required files recursively.

Could a similar approach be adopted in F# to avoid having to specify the order of files via the project file? Instead, the order could be implicitly imply the ordering of files based on the dependencies between the files. Given the current 'open' syntax, I would imagine this could work something like:

    open ``Helpers.fs``

If the F# file contains a single module then that module contents would be made available, and if the F# file was declared as a single namespace, then that namespace would simply be opened.

There could be some interesting edge cases if you wanted to specifically open one or more specific sub-modules or namespaces from within a file. It would also be interesting to hear if anyone else can think of any potential issues around a feature like this.

I would also imagine that this kind of referencing would also decrease the requirement to use a heavy-weight IDE such as Visual Studio to keep project files in sync.

## Alias Imports
The potential issue I raised in the previous section relating to referencing sub-modules or specific items in a namespace, led me back to the aliasing feature in C# which I used very occasionally. It would be interesting to know how feasible it would be to write something like:

    let foo = open some.long.namespace.structure.fooModule

Combining this with the previous idea would create an inter-file referencing system very similar to node or browserify, where your imported module or namespace is always allocated to a symbol rather than into the root scope of the whole file.

## Package Management
This leads me onto my final thought: package management.

NuGet is an invaluable tool that I've used on every project I've worked on. However, its integration with MSBuild, its powershell based tooling and the overhead of setting up packages (in comparison to more modern package managers which integrate directly with remote git repositories aka bower), leaves a lot to be desired for the developer experience.

What if there was some kind of way in which we could bake referencing third party assemblies directly into the F# language and compilation process and promote diversity in our package management options?

My first idea came before thinking through the aforementioned language features and utilised generative type providers to build a NuGet type provider. The type provider could take a package name and version specification and return some root class from the package. This approach might work for some packages, but could be fraught with issues such as:
- Packages with multiple namespaces
- Referencing the same package from multiple files
- Referencing multiple versions of the same package
- Resolution of dependencies – how do you coordinate shared dependencies between multiple packages?
- Would you need to use isolated app domains for each assembly referenced?

The second (and probably cleaner) option would to bake support directly into the language.

Much like type providers allow you to get types from anywhere, why not create some kind of assembly providers to allow you to create a provider to reference assemblies from a specific type of repository? Following the pattern of type providers, I'd guess the syntax might be nice to look something like:

    open Github<"fsharpfsharpx","1.2.3beta">

or

    open NuGet<"fsharpx","1.2.3">

Integrating this as a first class citizen of the F# compiler would allow simpler resolution of some of the issues listed above.
- Packages with multiple namespaces would simply be exposed to the root scope of that file, but could then have the specific namespaces "opened" on subsequent lines.
- Referencing the same package from multiple files could be simply duplicated during the compilation process.
- Referencing multiple versions of the same package (I think) should be possible given the way in which the 'extern' keyword works in C# (though I'm a little hazy on this)

## Closing
That's the end of my brain dump! I'd love to hear your thoughts and reasons why none of this would work ;) Here's the links to the feature requests I've opened for each of the things I've discussed:

- [Describe dependencies between files by extending 'open' syntax](https://fslang.uservoice.com/forums/245727-f-language/suggestions/6323146-describe-dependencies-between-files-by-extending)
- [Alias imported modules and namespaces](https://fslang.uservoice.com/forums/245727-f-language/suggestions/6323201-alias-imported-modules-and-namespaces)
- [Compiler integrated assembly imports from inside .fs files](https://fslang.uservoice.com/forums/245727-f-language/suggestions/6323264-compiler-integrated-assembly-imports-from-inside)

Existing feature suggestion for NuGet support in F# Interactive: [#package directive to import NuGet packages in F# interactive](https://fslang.uservoice.com/forums/245727-f-language/suggestions/5670137--package-directive-to-import-nuget-packages-in-f)
