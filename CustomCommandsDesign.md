# Custom Command for AOAPC-II Learning

## Basic Ideas

In order to organize the files more efficiently, I decided to write these shell scripts. They are designed only for this book, and if some of the functions could be used elsewhere, I think I'll abstract them later.



## Functions

### Add New Chapter

#### Function Design

Whenever I read a new chapter, there should be a new directory for it.

Each chapter directory contains several subdirectories:

* Notes - to store the notes recorded while reading the book
* Programs - to store the programs mentioned or practiced in the chapter
  * Examples - the examples shown in the chapter
    * cpp/java/python - Practice all of the programming languages
  * Exercises - the exercise questions left at the end of the chapter
    * cpp/java/python - Same as the above

#### Function Definition

**Name**: newchp

**Usage**: `newchp [-c <num>][-d "<dir1>, <dir2>, ..."]`

**Description**:

* `-c <num>`: Specify the chapter number. If omitted, deduce the next one

* `-d "<dir1>, <dir2>, ..."`: Apart from Notes and Programs directories, if other directories are needed, than list them as the parameters. They will be the subdirectories of Chapter root.

  Example: -d "resources/pic, resources/text, others"



### 