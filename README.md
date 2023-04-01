# Dollar Syntax Specification

**DollarDoc**

*March 2023*

## Introduction

The Dollar syntax is an object oriented way of writing documentation. Developed for documenting large software projects where references between files plays a large role in being able to maintain the documentation. The basic concept is having easily maintainable documentation files, where a file can be individually edited, and other files depending on a specific file will be automatically up to date. This is achieved by the depending files only containing references to the edited file. The depending files needs to be rebuilt, but not managed and edited individually. This idea has been further expanded with the use of plugins, to enable the use of a simple syntax to generate more complex output in the form of both text and images.

## Documentation Formats

The Dollar syntax was developed for use with Markdown, but technically the syntax does not specify which underlying documentation format is used. However, to write documentation in RST, AsciiDoc or even Latex, the format needs to be supported by the Dollar compiler. Plugins of Dollar is used to generate sections of documentation, and while the plugins themselves are agnostic to the format, their output will later be transformed to the desired format by the Dollar compiler.

## Files

The body of the documentation is meant to be written in a specific format. But this syntax changes how the files are supposed to be handled and viewed. A normal Markdown parser can not handle a Markdown Dollar file correctly and for this reason the files are using a different file ending, `.mdd`, for Markdown Dollar. There has been no decision made for exactly how to handle other formats, but a similar approach should be taken.

## Dollar Identity

Every Dollar file is identified with a Dollar ID. The ID namespace is global and ID's always need to be unique to be correctly referenced from other files in the project. The ID is allowed to contain alphanumeric characters and dashes. However, the ID must start and end with an alphanumeric character.

## Object Orientation

The Dollar syntax object orientation model builds on types. The types define a sort of interface for other objects to use. While this does not limit direct access to other objects, it helps structure the generation of content. The types can extend eachother, the base of any type is the dollar-type, noted `dollar`. This base type contains the `id` and `type` fields. The page-type, noted `page`, contains the `title` and `description` fields. Since page-type also extends dollar-type there are four fields in a page-type file: `id`, `type`, `title` and `description`.

The instances of types has no direct relation to eachother, other than the indirect relation of having the same type or a common type inheritance tree.

### Referencing Current Object

From within a Dollar file, it's possible to reference the containing object from itself. Similarly to how Python classes can reference themselves with `self` and Java classes with `this`. In Dollar you can reference the current object with `$this` and a header-content field some-key with `$this.some-key`.

## Header

The header is the first part of a Dollar file. The header starts on the first line of the document with three dashes, followed by the header content which is then ended with another three dashes. The header content is written in yaml.

***Example:***

``` text
---
id: some-id
type: dollar
---
```

### Dollar ID

The Dollar ID is a required field on any type, read the Dollar Identity chapter for more information.

### Dollar Type

Dollar type is a required field on any type, This tells the compiler how the file should be parsed and handled. For more information about types, read the Object Orientation chapter.

### Dollar Header References

The header is allowed to contain references to other fields inside the same object/file and to other objects/files. The fields in headers of other objects/files can not be used inside the header.

***Example:***

``` text
---
id: some-id
type: page

title: Some Title
description: Description for object with id $this.id
---
```

#### Trailing Chararcters on Header References

Sometimes a Dollar reference is the last word in a scentence, which should end with a dot. It is allowed for Dollar references to have trailing characters if they are followed by a space, newline character or is the last character in a field.

The allowed trailing characters are: `.`, `,`, `:`, `;`, `!`, `?`, `"`, `'`, `-` and `_`.

***Example:***

``` text
---
id: some-id
type: page

title: Some Title
description: Description for object with id $this.id.
---
```

## Content Body

The header is directly followed by the content body. The exact format depends on the chosen project text format. This document will assume this to be Markdown.

***Example:***

``` text
---
id: some-id
type: dollar
---

# This is a Normal Markdown Header

This is a normal markdown paragraph.
```

### Dollar Content References

Inside the content section of the Dollar file, there can exist two different types of Dollar references. A reference to a Dollar object or a reference to a field in a Dollar object.

#### Dollar Link References

When referencing a Dollar object directly, the generated output results in a link to that object/file.

***Example:***

``` text
Here is a link to some-id: $some-id
```

#### Dollar Data References

Access to either data from a field in the header of this object, or another object, is possible in the content-section. The referenced field needs to be of type string or number. If the field type is of something else, like a list, it needs to be passed through a Dollar function to render it properly. Read the Dollar Functions chapter for more information.

***Example:***

``` text
Description of this object: $this.description

Description of some-id: $some-id.description
```

#### Trailing Chararcters on Content References

Sometimes a Dollar reference is the last word in a scentence, which should end with a dot. It is allowed for Dollar references to have trailing characters if they are followed by a space or newline character.

The allowed trailing characters are: `.`, `,`, `:`, `;`, `!`, `?`, `"`, `'`, `-` and `_`.

***Example:***

``` text
This paragraph is ending with a link to $some-id.

This is ending with a data reference to $some-id.some-key.
```

### Dollar Functions

Dollar functions is used to display more complex datatypes and context from the Dollar project. The output from a function can be large, and be generated from context of multiple sources. It can also be functionality to format simple data in a more complex way than just Dollar references.

The best example of this is the Link and List functions in the Dollar standard library, read the Standard Library chapter for more information.

***Example:***

``` text
Some normal text.

$$Link($some-id)

Some more normal text.
```

### Dollar Blocks

Dollar blocks can be quite similar to Dollar functions in a way, but it brings with it a dynamic use of the Dollar syntax. Most projects need more than just plain text to document the project fully. Inside the block you are able to write multiple lines of text, including Dollar references. This is transformed into a section of formatted text, images or both.

Dollar standard library currently does not contain any Dollar blocks, but it's under revision and developent. Examples for this chapter will be provided upon releasing such standard library functionality.

### Escaping Characters

In most project documentation, at some point, there will exist the need to write a $-sign. It is possible to escape dollar syntax execution with a backslash.

***Example:***

``` text
This will not be a link: \$some-item
```

## Dollar Plugins

This specification has chosen not to decide on the chosen implementation language of plugins for a compiler. Technically this functionality can be done in any language and this chapter will simply explain the underlying functionality each plugin type brings to the table.

Plugins should be able to be written "on the fly" in a Dollar project. To extend functionality specific to a projects needs as it's being developed.

### Function Plugins

A function plugin is of a simple construct and should be easy to implement. It needs a name to be referenced from the documentation. It should provide information about what arguments it supports, such that a compiler can output proper error messages with suggestions. And it should implement the functionality needed to fill it's purpose. The output from the function should be agnostic to the underlying output format. It is not recommended that the function outputs a string of Markdown.

***Example:***

``` text
$$FunctionName($some-argument, Some string)
```

### Block Plugins

A block plugin enables the integration of other text syntaxes that can make use of the dollar reference syntax.

It's a complex plugin type to create, but designed to be easy to use while writing documentation.

It should implement the functionality needed to fill it's purpose. The output from the function should be agnostic to the underlying output format. It is not recommended that the block outputs a string of Markdown.

The content section of a block should always be indented with atleast four spaces.

***Example:***

``` text
$$$ BlockName
    someYaml: example
    $this.some-key: $this.some-value
$$$
```

### Extension Plugins

Extension plugins has the ability to create more types inside the documentation. Each type can be viewed as a `class` in traditional object oriented programming and each document using the type can be viewed as an `instance`.

#### Primary fields

***Example:***

``` text
---
id: some-a-id
type: some-type-a

some-type-a-field: Some value for primary field
---
```

#### Secondary fields

***Example:***

``` text
---
id: some-b-id
type: some-type-b

some-type-a-field: Some value for secondary field
---
```

#### Backreferences

Backreferencing is built in functionality to better relate objects/files to eachother by type. It's a uniform way to reference to another type, and for that referenced type to, for example, list those objects/files it's been referenced by.

Read the List References Function Plugin chapter for more information.

## Standard Library

### Link Function Plugin

The link function will print a link to an object of the page type. This will print the title of the page as a link, followed by the description of that page.

***Example:***

``` text
$$Link($some-id)
```

### List Function Plugin

The list function can print a list of strings and Dollar references.

***Example:***

``` text
---
id: some-id
type: some-type

list:
    - Some string
    - $this.id
    - $some-other-item
---

$$List($this.list)
```

### List References Function Plugin

The list references function is used to list backreferences on a specific object/file. read the Backreferences chapter for more information.

***Example:***

``` text
---
id: some-b-id
type: some-type-b

some-type-a-backref-field: $some-a-id
---
```

``` text
---
id: some-a-id
type: some-type-a
---

$$ListRef($this)
```

### Page Extension Plugin

The page extension is a core extension plugin. This type adds two required fields to the header-section, `title` and `description`. Functionality built around these fields are meant to enrich the documentation and make it easier to follow for any reader.

***Example:***

``` text
---
id: some-page
type: page

title: Some Title
description: A description for the page.
---
```
