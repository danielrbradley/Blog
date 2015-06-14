@{
    Layout = "post";
    Title = "Better MVC Enum Dropdown List Implementation";
    Date = "2012-06-21T12:10:00";
    Tags = "aspnet csharp";
    Description = "Created a new ASP.NET MVC extension method for creating a “DropDownListFor” an Enum or nullable enum.";
}

Created a new ASP.NET MVC extension method for creating a “DropDownListFor” an Enum or nullable enum.

Features:
- Automatic support for nullable Enum property types.
- Automatic conversion of camel cased enumeration name to human readable name.
- Supports custom name generation for enumeration values.
- Auto-orders options by name.

If you want to add a feature that’s not there, feel free to fork the gist below and go crazy! (let me know and I’ll update it here too!)

<script src="https://gist.github.com/danielrbradley/2965119.js"></script>
