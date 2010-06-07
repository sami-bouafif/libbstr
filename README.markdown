libbstr : Binary String support library.
========================================
Binary String support — Typedef and functions to support binary
strings.

## Synopsis

    #include <bstr.h>
    
    typedef             bstr_t;
    bstr_t              bstr_new                            (char *from,
                                                             size_t size);
    bstr_t              bstr_newFromCStr                    (char *cstr);
    void                bstr_free                           (bstr_t bstr);
    char*               bstr_toCStr                         (bstr_t bstr);
    size_t              bstr_len                            (bstr_t bstr);
    bstr_t              bstr_cat                            (bstr_t bstr,
                                                             char *cstr,
                                                             size_t size);
    bstr_t              bstr_catBStr                        (bstr_t to,
                                                             bstr_t from);
    bstr_t              bstr_catCStr                        (bstr_t bstr,
                                                             char *cstr);
    bstr_t              bstr_dup                            (bstr_t bstr);
    int                 bstr_asprintf                       (bstr_t *bstr,
                                                             char *fmt,
                                                             ...);
    int                 bstr_scatprintf                     (bstr_t *bstr,
                                                             char *fmt,
                                                             ...);

## Description

A collection of function and a structure to support strings that
can contain one or more `\0` characters. These functions implements
basic string operations similar to these applicable to standard
string format.

## Details

### bstr\_t

An opaque structure to represent a bstring. It is a rough
representation of pascal string where the length of the string
precede the string itself, so `\0` characters can be embedded in
the string without truncating it.

The memory occupied by
`bstr_t` is represented internally as `size_t` followed by a
 `char *` and a `\0` character, and
`bstr_t` itself points to the `char *` portion. This is set manually rather
than using a `struct` to emphasis the fact that the only
exploitable part is the string part. The length part is updated
automatically by `bstr_t` related functions.

>***
>_**Note**_   
>Using `struct` to define `bstr_t` is just a way to prevent the
>compiler from implicit casting `bstr_t` to `char *`.

>***

Since
`bstr_t` points to the string part of the structure, it can be used anywhere
`char *` is usable (with an explicit cast) but, if the string has
more than one `\0` character, the result will be truncated on the
first one.

Note also that modifying the string part of
`bstr_t`(modifying `bstr_t` as a `char *`) will invalidate the 
length part and therefore, the effect
of `bstr_t` related functions on the modified version will be unpredictable.  
It is advised to use the cast in read-only operations. For
read-write ones, get a copy of the string with
`bstr_toCStr()`.


* * * * *

### bstr\_new ()

    bstr_t              bstr_new                            (char *from,
                                                             size_t size);

Create a new bstring of length *`size`* from the character array
*`from`*.

If *`size`* is -1, the length of the resulting bstring is
calculated using `strlen(`*`from`*`)` and the
resulting bstring will be a copy of *`from`*.  
If *`size`* is `0`, the length of the resulting bstring will be `0` and
*`from`* will be ignored.  
If *`from`* is `NULL`, `bstr_new()` will create a bstring of length
*`size`*
 with zeroed elements.

>***
>_**Note**_   
>If *`size`* exceeds the length of the character array *`from`*, the
>result will be unpredictable.

>***

*`from`*: a string from which to create the bstring or `NULL`.   
*`size`*: number of characters to use in creating the bstring or -1.   
*Returns*: a newly allocated bstring or `NULL` on error (usually a result of
memory allocation error). The returned value should be freed with
`bstr_free()` when no longer needed.

* * * * *

### bstr\_newFromCStr ()

    bstr_t              bstr_newFromCStr                    (char *cstr);

Construct a new bstring from a C string. It is equivalent to
`bstr_new(`*`cstr`*`, -1)` (See also `bstr_new()`.

If *`cstr`* is `NULL`, the length of the resulting bstring will be
`0`.

*`cstr`*: a C-style string or `NULL`.  
*Returns*: a newly allocated bstring copy of *`cstr`* or `NULL` on error
(usually a result of memory allocation error). The returned value
should be freed with `bstr_free()` when no longer needed.

* * * * *

### bstr\_free ()

    void                bstr_free                           (bstr_t bstr);

Free the memory allocated to *`bstr`*.

>***
>_**Note**_   
>It is advised to free memory of vars that are no longer used with `bstr_free()`

>***

*`bstr`*: the bstring to free.

* * * * *

### bstr\_toCStr ()

    char*               bstr_toCStr                         (bstr_t bstr);

Convert a bstring to a C-style string.

The resulting string will be truncated at the first `\0`. So if
*`bstr`* doesn't contain any `\0` (only the `\0` at the end), the
resulting string should be a copy of the original.

Note that *`bstr`* can be used as `char *` with an explicit cast
like:

    str = (char *)bstr;

but there is some rules on using *`str`* in this case:  
- *`str`* should never be freed with `free()`.  
- *`str`* should never be manipulated with standard string
functions, get a C-style copy of *`bstr`* with `bstr_toCStr()` instead.

The cast to `char *` can be used for simple operation such as
printing :

    printf("%s\n",(char *)bstr);

and even this is not needed since the `%B` modifier can be used to
print a bstring.

*`bstr`*: a bstring to convert.   
*Returns*: a C-style string or `NULL` on error (usually a memory
allocation error). The returned string should be freed with
`free()` when no longer needed.

* * * * *

### bstr\_len ()

    size_t              bstr_len                            (bstr_t bstr);

Retrieve the length of a bstring.

*`bstr`*: a bstring.   
*Returns*: the length of *`bstr`*.

* * * * *

### bstr\_cat ()

    bstr_t              bstr_cat                            (bstr_t bstr,
                                                             char *cstr,
                                                             size_t size);

Concatenate *`size`* characters from *`cstr`* to *`bstr`*.

If *`size`* is -1, `strlen()` is used to
calculate the number of characters to concatenate.  
If *`cstr`* is `NULL`, *`bstr`* is return as it is.  
If *`bstr`* is `NULL`, this is the same as calling
`bstr_new()` with *`cstr`* and *`size`*.

*`bstr`*: the bstring to append to or `NULL`.   
*`cstr`*: the C-style string to append or `NULL`.   
*`size`*: the number of characters to append or -1.   
*Returns*: the concatenated bstring (*`bstr`*) or `NULL` on error.
(usually a result of a memory allocation error). The returned 
value should be freed with `bstr_free()` when no longer needed.

* * * * *

### bstr\_catBStr ()

    bstr_t              bstr_catBStr                        (bstr_t to,
                                                             bstr_t from);

Concatenate 2 bstrings. The result is stored in *`to`*.

If *`from`* is `NULL`, *`to`* remain unchanged and returned as it
is.  
If *`to`* is `NULL`, this is equivalent to `bstr_dup(`*`from`*`)`.

>***
>_**Note**_   
>If *`to`* or *`from`* are not initialized, the result is
>unpredictable.

>***

*`to`*: the bstring to append to or `NULL`.   
*`from`*: the bstring to append or `NULL`.   
*Returns*: the concatenated bstring (*`to`*) or `NULL` on error (usually a
result of an error while expanding the memory allocated to *`to`*).
The returned value should be freed with`bstr_free()` when no longer needed.

* * * * *

### bstr\_catCStr ()

    bstr_t              bstr_catCStr                        (bstr_t bstr,
                                                             char *cstr);

Concatenate *`cstr`* to *`bstr`*. This has the same effect as
`bstr_cat(`*`bstr`*`,` *`cstr`*, `-1)` (See `bstr_cat()`).

*`bstr`*: the bstring to append to.   
*`cstr`*: the C-style string to append.   
*Returns*: the concatenated bstring (*`bstr`*) or `NULL` on error (memory
allocation error). The returned value should be freed with`bstr_free()`
when no longer needed.

* * * * *

### bstr\_dup ()

    bstr_t              bstr_dup                            (bstr_t bstr);

Duplicates a bstring and return the newly created one.

*`bstr`*: the bstring to duplicate.   
*Returns*: a bstring copy of *`bstr`* or `NULL` on error (memory allocation
error). The returned value should be freed with `bstr_free()`
when no longer needed.

* * * * *

### bstr\_asprintf ()

    int                 bstr_asprintf                       (bstr_t *bstr,
                                                             char *fmt,
                                                             ...);

Like asprintf, but prints in a bstr\_t var. No need to preallocate
storage, but *`bstr`* must be freed manually.

>***
>_**Note**_   
>There is also the new printf modifier `%B` that can be used to
>print a bstring. This modifier is usable in all printf-like
>functions.

>***

*`bstr`*: the bstring to print to.   
*`fmt`*: the printf-style format string.   
*`...`*: the list of args conforming to *`fmt`* format.   
*Returns*: the number of character writen or -1 on error. *`bstr`* should be
freed with `bstr_free()` when no longer needed.

* * * * *

### bstr\_scatprintf ()

    int                 bstr_scatprintf                     (bstr_t *bstr,
                                                             char *fmt,
                                                             ...);

Like `bstr_asprintf()`, but append to the input bstr\_t var. *`bstr`* 
is resized to fit to the concatenated string.


>***
>_**Note**_   
>There is also the new printf modifier `%B` that can be used to
>print a bstring. This modifier is usable in all printf-like
>functions.

>***


*`bstr`*: the bstring to append to.   
*`fmt`*: the printf-style format string.   
*`...`*: the list of args conforming to *`fmt`* format.   
*Returns*: the number of appended characters or -1 on error. *`bstr`* should
be freed with `bstr_free()` when no longer needed.

* * * * *
