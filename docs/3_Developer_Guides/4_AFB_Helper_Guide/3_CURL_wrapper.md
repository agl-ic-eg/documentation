---
title: CURL wrapping functions
---

## int curl_wrap_perform (CURL * curl, char **result, size_t * size)

Perform the CURL operation for 'curl' and put the result in memory. If 'result'
isn't NULL it receives the returned content that then must be freed. If 'size'
isn't NULL, it receives the size of the returned content. Note that if not NULL,
the real content is one byte greater than the read size and the last byte
zero. This facility allows to handle the returned content as a null terminated
C-string.

## void curl_wrap_do(CURL *curl, void (*callback)(void *closure, int status, CURL *curl, const char *result, size_t size), void *closure)

Will perform the CURL operation and on success invokes the callback with the
result and size of the operation or error on a failure.

## int curl_wrap_content_type_is (CURL * curl, const char *value)

Request `Content-Type` information from the curl session with this function

Returns non-zero value if the CURL content type match the `value` and 0 otherwize

## long curl_wrap_response_code_get(CURL *curl)

Request the response code to a CURL operation.

Returns the response code received on success or 0 on failure.

## CURL *curl_wrap_prepare_get_url(const char *url)

Prepare a GET CURL operation on the url given as parameter and Returns the CURL
pointer.

## CURL *curl_wrap_prepare_get(const char *base, const char *path, const char * const *args)

Prepare a GET CURL operation on the decomposed url and escape it. The `url` has been
decomposed in 3 parts:

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `args`: optionnal array of arguments provided for the GET request.

Returns the prepared CURL request.

## int curl_wrap_add_header(CURL *curl, const char *header)

Add a header to a CURL operation.

Returns the prepared CURL request.

## int curl_wrap_add_header_value(CURL *curl, const char *name, const char *value)

Add a tuple `name`, `value` to the header of a CURL operation.

Returns the prepared CURL request.

## CURL *curl_wrap_prepare_post_url_data(const char *url, const char *datatype, const char *data, size_t szdata)

Prepare a POST CURL operation on the provided `url`.

* `url`: a HTTP url.
* `datatype`: HTTP `Content-Type` to use for the operation.
* `data`: data to send.
* `szdata`: size of the data to send.

Returns the prepared CURL request.

## CURL *curl_wrap_prepare_post_simple_unescaped(const char *base, const char *path, const char *args)

Prepare a POST CURL operation on an unescaped `url` with arguments provided as
a simple string.

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `args`: optionnals argument for the POST http request.

Returns the prepared CURL request.

## CURL *curl_wrap_prepare_post_simple_escaped(const char *base, const char *path, char *args)

Prepare a POST CURL operation on an escaped `url` with arguments provided as
a simple string.

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `args`: optionnals argument for the POST http request.

Returns the prepared CURL request.

## CURL *curl_wrap_prepare_post_unescaped(const char *base, const char *path, const char *separator, const char * const *args)

Prepare a POST CURL operation on an unescaped `url` with arguments provided as
an array of string concatened with a provided separator string.

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `separator`: string used as a separator when concatening the arguments.
* `args`: optionnal array of arguments for the POST http request.

Returns the prepared CURL request.

## CURL *curl_wrap_prepare_post_escaped(const char *base, const char *path, const char * const *args)

Prepare a POST CURL operation on an unescaped `url` with arguments provided as
an array of string concatened without any separator.

* `base`: representing the FQDN of the url.
* `path`: the path to the requested page.
* `args`: optionnal array of arguments for the POST http request.

Returns the prepared CURL request.