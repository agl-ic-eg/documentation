---
title: Escaping helpers functions
---

## char *escape_url(const char *base, const char *path, const char * const *args, size_t *length)

Escape an `url` and `arguments` and returned it as a string.

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `args`: optionnal array of arguments provided for the GET request.
* `length`: length of the returned `url`.

Returns the escaped `url`.

## const char *escape_args(const char * const *args, size_t *length)

Escape an array of arguments and returned the lenght of the escaped arguments
string.

* `args`: array of arguments provided for the GET request.
* `length`: length of the returned `arguments`.

Returns the escaped `arguments`.

## const char *escape_str(const char *str, size_t *length)

Escape a string and returns it.

* `str`: the string to escape.
* `length`: length of the returned string.

Returns the escaped string.

## const char **unescape_args(const char *args)

Unescape an argument and returns it.

* `args`: the argument to unescape.
