---
title: Binder References
---

i. TYPES AND GLOBALS
======================

## The global afbBindingRoot

The global **afbBindingRoot** of type **afb_api_t** is always implicitly
defined for bindings of version 3 or upper. It records the root api of
the binding.

When the binding has a defined **afbBindingExport**,  the root api
**afbBindingRoot** is the **afb_pi_t** relative to the api created for
this static description.

When the binding has no defined **afbBindingExport**, the root api is
a virtual api representing the shared object of the binding. In that case
the name of the api is the path of the shared object. Its use is restricted
but allows log messages.

## The global afbBindingExport

The global **afbBindingExport** is not mandatory.

If **afbBindingExport** is defined and exported, it must be of the type
**const afb_binding_t** and must describe the *root* api of the binding.

## The type afb_api_t

Bindings now can declare more than one api. The counter part is that
a new handle is needed to manage apis. These handles are of the type
**afb_api_t**.

It is defined as below.

```C
typedef struct afb_api_x3 afb_api_t;
```

## The type afb_binding_t

The main structure, of type **afb_binding_t**, for describing the binding
must be exported under the name **afbBindingExport**.

This structure is defined as below.

```C
typedef struct afb_binding_v3 afb_binding_t;
```

Where:

```C
/**
 * Description of the bindings of type version 3
 */
struct afb_binding_v3
{
	/** api name for the binding, can't be NULL */
	const char *api;

	/** textual specification of the binding, can be NULL */
	const char *specification;

	/** some info about the api, can be NULL */
	const char *info;

	/** array of descriptions of verbs terminated by a NULL name, can be NULL */
	const struct afb_verb_v3 *verbs;

	/** callback at load of the binding */
	int (*preinit)(struct afb_api_x3 *api);

	/** callback for starting the service */
	int (*init)(struct afb_api_x3 *api);

	/** callback for handling events */
	void (*onevent)(struct afb_api_x3 *api, const char *event, struct json_object *object);

	/** userdata for afb_api_x3 */
	void *userdata;

	/** space separated list of provided class(es) */
	const char *provide_class;

	/** space separated list of required class(es) */
	const char *require_class;

	/** space separated list of required API(es) */
	const char *require_api;

	/** avoids concurrent requests to verbs */
	unsigned noconcurrency: 1;
};
```

## The type afb_verb_t

Each verb is described with a structure of type **afb_verb_t**
defined below:

```C
typedef struct afb_verb_v3 afb_verb_t;
```

```C
/**
 * Description of one verb as provided for binding API version 3
 */
struct afb_verb_v3
{
	/** name of the verb, NULL only at end of the array */
	const char *verb;

	/** callback function implementing the verb */
	void (*callback)(afb_req_t_x2 *req);

	/** required authorization, can be NULL */
	const struct afb_auth *auth;

	/** some info about the verb, can be NULL */
	const char *info;

	/**< data for the verb callback */
	void *vcbdata;

	/** authorization and session requirements of the verb */
	uint16_t session;

	/** is the verb glob name */
	uint16_t glob: 1;
};
```

The **session** flags is one of the constant defined below:

| Name                   | Description
|:----------------------:|------------------------------------------------------
| AFB_SESSION_NONE       | no flag, synonym to 0
| AFB_SESSION_LOA_0      | Requires the LOA to be 0 or more, synonym to 0 or AFB_SESSION_NONE
| AFB_SESSION_LOA_1      | Requires the LOA to be 1 or more
| AFB_SESSION_LOA_2      | Requires the LOA to be 2 or more
| AFB_SESSION_LOA_3      | Requires the LOA to be 3 or more
| AFB_SESSION_CHECK      | Requires the token to be set and valid
| AFB_SESSION_REFRESH    | Implies a token refresh
| AFB_SESSION_CLOSE      | Implies closing the session after request processed

The LOA (Level Of Assurance) is set, by binding api, using the function **afb_req_session_set_LOA**.

The session can be closed, by binding api, using the function **afb_req_session_close**.

## The types afb_auth_t and afb_auth_type_t

The structure **afb_auth_t** is used within verb description to
set security requirements.
The interpretation of the structure depends on the value of the field **type**.

```C
typedef struct afb_auth afb_auth_t;

/**
 * Definition of an authorization entry
 */
struct afb_auth
{
	/** type of entry @see afb_auth_type */
	enum afb_auth_type type;

	union {
		/** text when @ref type == @ref afb_auth_Permission */
		const char *text;

		/** level of assurancy when @ref type ==  @ref afb_auth_LOA */
		unsigned loa;

		/** first child when @ref type in { @ref afb_auth_Or, @ref afb_auth_And, @ref afb_auth_Not } */
		const struct afb_auth *first;
	};

	/** second child when @ref type in { @ref afb_auth_Or, @ref afb_auth_And } */
	const struct afb_auth *next;
};

```

The possible values for **type** is defined here:

```C
typedef enum afb_auth_type afb_auth_type_t;

/**
 * Enumeration  for authority (Session/Token/Assurance) definitions.
 *
 * @see afb_auth
 */
enum afb_auth_type
{
	/** never authorized, no data */
	afb_auth_No = 0,

	/** authorized if token valid, no data */
	afb_auth_Token,

	/** authorized if LOA greater than or equal to data 'loa' */
	afb_auth_LOA,

	/** authorized if permission 'text' is granted */
	afb_auth_Permission,

	/** authorized if 'first' or 'next' is authorized */
	afb_auth_Or,

	/** authorized if 'first' and 'next' are authorized */
	afb_auth_And,

	/** authorized if 'first' is not authorized */
	afb_auth_Not,

	/** always authorized, no data */
	afb_auth_Yes
};
```

Example:

```C
static const afb_auth_t myauth[] = {
    { .type = afb_auth_Permission, .text = "urn:AGL:permission:me:public:set" },
    { .type = afb_auth_Permission, .text = "urn:AGL:permission:me:public:get" },
    { .type = afb_auth_Or, .first = &myauth[1], .next = &myauth[0] }
};
```


## The type afb_req_subcall_flags_t

This is an enumeration that defines bit's positions for setting behaviour
of subcalls.

| flag                       | value | description
|----------------------------|-------|--------------
| afb_req_subcall_catch_events | 1 | the calling API wants to receive the events from subscription
| afb_req_subcall_pass_events  | 2 | the original request will receive the events from subscription
| afb_req_subcall_on_behalf    | 4 | the calling API wants to use the credentials of the original request
| afb_req_subcall_api_session  | 8 | the calling API wants to use its session instead of the one of the original request

ii. MACROS FOR LOGGING
=================

The final behaviour of macros can be tuned using 2 defines that must be defined
before including **<afb/afb-binding.h>**.

| define                                | action
|---------------------------------------|--------------------
| AFB_BINDING_PRAGMA_NO_VERBOSE_DATA    | show file and line, remove function and text message
| AFB_BINDING_PRAGMA_NO_VERBOSE_DETAILS | show text, remove function, line and file

## Logging for an api

The following macros must be used for logging for an **api** of type
**afb_api_t**.

```C
AFB_API_ERROR(api,fmt,...)
AFB_API_WARNING(api,fmt,...)
AFB_API_NOTICE(api,fmt,...)
AFB_API_INFO(api,fmt,...)
AFB_API_DEBUG(api,fmt,...)
```

## Logging for a request


The following macros can be used for logging in the context
of a request **req** of type **afb_req_t**:

```C
AFB_REQ_ERROR(req,fmt,...)
AFB_REQ_WARNING(req,fmt,...)
AFB_REQ_NOTICE(req,fmt,...)
AFB_REQ_INFO(req,fmt,...)
AFB_REQ_DEBUG(req,fmt,...)
```

By default, the logging macros add file, line and function
indication.

## Logging legacy

The following macros are provided for legacy.

```C
AFB_ERROR(fmt,...)
AFB_WARNING(fmt,...)
AFB_NOTICE(fmt,...)
AFB_INFO(fmt,...)
AFB_DEBUG(fmt,...)
```

iii. FUNCTIONS OF CLASS **afb_api**
============================

## General functions

### afb_api_name

```C
/**
 * Get the name of the 'api'.
 *
 * @param api the api whose name is to be returned
 *
 * @return the name of the api.
 *
 * The returned value must not be changed nor freed.
 */
const char *afb_api_name(
			afb_api_t api);
```

### afb_api_get_userdata

```C
/**
 * Get the "userdata" pointer of the 'api'
 *
 * @param api the api whose user's data is to be returned
 *
 * @return the user's data  pointer of the api.
 *
 * @see afb_api_set_userdata
 */
void *afb_api_get_userdata(
			afb_api_t api);
```

### afb_api_set_userdata

```C
/**
 * Set the "userdata" pointer of the 'api' to 'value'
 *
 * @param api   the api whose user's data is to be set
 * @param value the data to set
 *
 * @see afb_api_get_userdata
 */
void afb_api_set_userdata(
			afb_api_t api,
			void *value);
```

### afb_api_require_api

```C
/**
 * Check that it requires the API of 'name'.
 * If 'initialized' is not zero it requests the API to be
 * initialized, implying its initialization if needed.
 *
 * Calling this function is only allowed within init.
 *
 * A single request allows to require multiple apis.
 *
 * @param api the api that requires the other api by its name
 * @param name a space separated list of required api names
 * @param initialized if zero, the api is just required to exist. If not zero,
 * the api is required to exist and to be initialized at return of the call
 * (initializing it if needed and possible as a side effect of the call).
 *
 * @return 0 in case of success or -1 in case of error with errno set appropriately.
 */
int afb_api_require_api(
			afb_api_t api,
			const char *name,
			int initialized);
```


## Verbosity functions

### afb_api_wants_log_level

```C
/**
 * Is the log message of 'level (as defined for syslog) required for the api?
 *
 * @param api   the api to check
 * @param level the level to check as defined for syslog:
 *
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @return 0 if not required or a value not null if required
 *
 * @see syslog
 */
int afb_api_wants_log_level(
			afb_api_t api,
			int level);
```

### afb_api_vverbose

```C
/**
 * Send to the journal with the logging 'level' a message described
 * by 'fmt' applied to the va-list 'args'.
 *
 * 'file', 'line' and 'func' are indicators of code position in source files
 * (see macros __FILE__, __LINE__ and __func__).
 *
 * 'level' is defined by syslog standard:
 *
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @param api the api that collects the logging message
 * @param level the level of the message
 * @param file the source file that logs the messages or NULL
 * @param line the line in the source file that logs the message
 * @param func the name of the function in the source file that logs
 * @param fmt the format of the message as in printf
 * @param args the arguments to the format string of the message as a va_list
 *
 * @see syslog
 * @see printf
 */
void afb_api_vverbose(
			afb_api_t api,
			int level,
			const char *file,
			int line,
			const char *func,
			const char *fmt,
			va_list args);
```

### afb_api_verbose

```C
/**
 * Send to the journal with the log 'level' a message described
 * by 'fmt' and following parameters.
 *
 * 'file', 'line' and 'func' are indicators of position of the code in source files
 * (see macros __FILE__, __LINE__ and __func__).
 *
 * 'level' is defined by syslog standard:
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @param api the api that collects the logging message
 * @param level the level of the message
 * @param file the source file that logs the messages or NULL
 * @param line the line in the source file that logs the message
 * @param func the name of the function in the source file that logs
 * @param fmt the format of the message as in printf
 * @param ... the arguments to the format string of the message
 *
 * @see syslog
 * @see printf
 */
void afb_api_verbose(
			afb_api_t api,
			int level,
			const char *file,
			int line,
			const char *func,
			const char *fmt,
			...);
```

## Data retrieval functions

### afb_api_rootdir_get_fd

```C
/**
 * Get the root directory file descriptor. This file descriptor can
 * be used with functions 'openat', 'fstatat', ...
 *
 * CAUTION, manipulate this descriptor with care, in particular, don't close
 * it.
 *
 * This can be used to get the path of the root directory using:
 *
 * char buffer[MAX_PATH], proc[100];
 * int dirfd = afb_api_rootdir_get_fd(api);
 * snprintf(proc, sizeof proc, "/proc/self/fd/%d", dirfd);
 * readlink(proc, buffer, sizeof buffer);
 *
 * But note that within AGL this is the value given by the environment variable
 * AFM_APP_INSTALL_DIR.
 *
 * @param api the api that uses the directory file descriptor
 *
 * @return the file descriptor of the root directory.
 *
 * @see afb_api_rootdir_open_locale
 */
int afb_api_rootdir_get_fd(
			afb_api_t api);
```

### afb_api_rootdir_open_locale

```C
/**
 * Opens 'filename' within the root directory with 'flags' (see function openat)
 * using the 'locale' definition (example: "jp,en-US") that can be NULL.
 *
 * The filename must be relative to the root of the bindings.
 *
 * The opening mode must be for read or write but not for O_CREAT.
 *
 * The definition of locales and of how files are searched can be checked
 * here: https://www.w3.org/TR/widgets/#folder-based-localization
 * and https://tools.ietf.org/html/rfc7231#section-5.3.5
 *
 * @param api the api that queries the file
 * @param filename the relative path to the file to open
 * @param flags the flags for opening as for C function 'open'
 * @param locale string indicating how to search content, can be NULL
 *
 * @return the file descriptor or -1 in case of error and errno is set with the
 * error indication.
 *
 * @see open
 * @see afb_api_rootdir_get_fd
 */
int afb_api_rootdir_open_locale(
			afb_api_t api,
			const char *filename,
			int flags,
			const char *locale);
```

### afb_api_settings

```C
/**
 * Settings of the api.
 *
 * Get the settings of the API. The settings are recorded
 * as a JSON object. The returned object should not be modified.
 * It MUST NOT be released using json_object_put.
 *
 * @param api the api whose settings are required
 *
 * @returns The setting object.
 */
struct json_object *afb_api_settings(
			struct afb_api_x3 *api);
```

## Calls and job functions

### afb_api_call

```C
/**
 * Calls the 'verb' of the 'apiname' with the arguments 'args' and 'verb' in the name of the binding 'api'.
 * The result of the call is delivered to the 'callback' function with the 'callback_closure'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * The 'callback' receives 5 arguments:
 *  1. 'closure' the user defined closure pointer 'closure',
 *  2. 'object'  a JSON object returned (can be NULL)
 *  3. 'error'   a string not NULL in case of error but NULL on success
 *  4. 'info'    a string handling some info (can be NULL)
 *  5. 'api'     the api
 *
 * NOTE: For convenience, *json_object_put* is called on 'object' after the
 * callback returns. So, it is wrong to call *json_object_put* in the callback.
 *
 * @param api      The api that makes the call
 * @param apiname  The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param callback The to call on completion
 * @param closure  The closure to pass to the callback
 *
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 * @see afb_api_call_sync
 */
void afb_api_call(
			afb_api_t api,
			const char *apiname,
			const char *verb,
			struct json_object *args,
			void (*callback)(
					void *closure,
					struct json_object *object,
					const char *error,
					const char * info,
					afb_api_t api),
			void *closure);
```

### afb_api_call_sync

```C
/**
 * Calls the 'verb' of the 'apiname' with the arguments 'args' and 'verb' in the name of the binding 'api'.
 * 'result' will receive the response.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param api      The api that makes the call
 * @param apiname  The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param object   Where to store the returned object - should call json_object_put on it - can be NULL
 * @param error    Where to store the copied returned error - should call free on it - can be NULL
 * @param info     Where to store the copied returned info - should call free on it - can be NULL
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 * @see afb_api_call
 */
int afb_api_call_sync(
			afb_api_t api,
			const char *apiname,
			const char *verb,
			struct json_object *args,
			struct json_object **object,
			char **error,
			char **info);
```

### afb_api_queue_job

```C
/**
 * Queue the job defined by 'callback' and 'argument' for being executed asynchronously
 * in this thread (later) or in an other thread.
 *
 * If 'group' is not NULL, the jobs queued with a same value (as the pointer value 'group')
 * are executed in sequence in the order of there submission.
 *
 * If 'timeout' is not 0, it represent the maximum execution time for the job in seconds.
 * At first, the job is called with 0 as signum and the given argument.
 *
 * The job is executed with the monitoring of its time and some signals like SIGSEGV and
 * SIGFPE. When a such signal is catched, the job is terminated and reexecuted but with
 * signum being the signal number (SIGALRM when timeout expired).
 *
 * When executed, the callback function receives 2 arguments:
 *
 *  - int signum: the signal catched if any or zero at the beginning
 *  - void *arg: the parameter 'argument'
 *
 * A typical implementation of the job callback is:
 *
 * void my_job_cb(int signum, void *arg)
 * {
 *	struct myarg_t *myarg = arg;
 *	if (signum)
 *		AFB_API_ERROR(myarg->api, "job interrupted with signal %s", strsignal(signum));
 *	else
 *		really_do_my_job(myarg);
 * }
 *
 * @param api the api that queue the job
 * @param callback the job as a callback function
 * @param argument the argument to pass to the queued job
 * @param group the group of the job, NULL if no group
 * @param timeout the timeout of execution of the job
 *
 * @return 0 in case of success or -1 in case of error with errno set appropriately.
 */
int afb_api_queue_job(
			afb_api_t api,
			void (*callback)(int signum, void *arg),
			void *argument,
			void *group,
			int timeout);
```

## Event functions

### afb_api_broadcast_event

```C
/**
 * Broadcasts widely the event of 'name' with the data 'object'.
 * 'object' can be NULL.
 *
 * For convenience, the function calls 'json_object_put' for 'object'.
 * Thus, in the case where 'object' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * Calling this function is only forbidden during preinit.
 *
 * The event sent as the name API/name where API is the name of the
 * api.
 *
 * @param api the api that broadcast the event
 * @param name the event name suffix
 * @param object the object that comes with the event
 *
 * @return 0 in case of success or -1 in case of error
 */
int afb_api_broadcast_event(
			afb_api_t api,
			const char *name,
			struct json_object *object);
```

### afb_api_make_event

```C
/**
 * Creates an event of 'name' and returns it.
 *
 * Calling this function is only forbidden during preinit.
 *
 * See afb_event_is_valid to check if there is an error.
 *
 * The event created as the name API/name where API is the name of the
 * api.
 *
 * @param api the api that creates the event
 * @param name the event name suffix
 *
 * @return the created event. Use the function afb_event_is_valid to check
 * whether the event is valid (created) or not (error as reported by errno).
 *
 * @see afb_event_is_valid
 */
afb_event_t afb_api_make_event(
			afb_api_t api,
			const char *name);
```

### afb_api_event_handler_add

```C
/**
 * Add a specific event handler for the api
 *
 * The handler callback is called when an event matching the given pattern
 * is received (it is received if broadcasted or after subscription through
 * a call or a subcall).
 *
 * The handler callback receive 4 arguments:
 *
 *  - the closure given here
 *  - the event full name
 *  - the companion JSON object of the event
 *  - the api that subscribed the event
 *
 * @param api the api that creates the handler
 * @param pattern the global pattern of the event to handle
 * @param callback the handler callback function
 * @param closure the closure of the handler
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_api_on_event
 * @see afb_api_event_handler_del
 */
int afb_api_event_handler_add(
			afb_api_t api,
			const char *pattern,
			void (*callback)(
					void *,
					const char*,
					struct json_object*,
					afb_api_t),
			void *closure);
```

### afb_api_event_handler_del

```C
/**
 * Delete a specific event handler for the api
 *
 * @param api the api that the handler belongs to
 * @param pattern the global pattern of the handler to remove
 * @param closure if not NULL it will receive the closure set to the handler
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_api_on_event
 * @see afb_api_event_handler_add
 */
int afb_api_event_handler_del(
			afb_api_t api,
			const char *pattern,
			void **closure);

```

## Systemd functions

### afb_api_get_event_loop

```C
/**
 * Retrieves the common systemd's event loop of AFB
 *
 * @param api the api that uses the event loop
 *
 * @return the systemd event loop if active, NULL otherwise
 *
 * @see afb_api_get_user_bus
 * @see afb_api_get_system_bus
 */
struct sd_event *afb_api_get_event_loop(
			afb_api_t api);
```

### afb_api_get_user_bus

```C
/**
 * Retrieves the common systemd's user/session d-bus of AFB
 *
 * @param api the api that uses the user dbus
 *
 * @return the systemd user connection to dbus if active, NULL otherwise
 *
 * @see afb_api_get_event_loop
 * @see afb_api_get_system_bus
 */
struct sd_bus *afb_api_get_user_bus(
			afb_api_t api);
```

### afb_api_get_system_bus

```C
/**
 * Retrieves the common systemd's system d-bus of AFB
 *
 * @param api the api that uses the system dbus
 *
 * @return the systemd system connection to dbus if active, NULL otherwise
 *
 * @see afb_api_get_event_loop
 * @see afb_api_get_user_bus
 */
struct sd_bus *afb_api_get_system_bus(
			afb_api_t api);
```


## Dynamic api functions

### afb_api_new_api

```C
/**
 * Creates a new api of name 'apiname' briefly described by 'info' (that can
 * be NULL).
 *
 * When the pre-initialization function is given, it is a function that
 * receives 2 parameters:
 *
 *  - the closure as given in the call
 *  - the created api that can be initialised
 *
 * This pre-initialization function must return a negative value to abort
 * the creation of the api. Otherwise, it returns a non-negative value to
 * continue.
 *
 * @param api the api that creates the other one
 * @param apiname the name of the new api
 * @param info the brief description of the new api (can be NULL)
 * @param noconcurrency zero or not zero whether the new api is reentrant or not
 * @param preinit the pre-initialization function if any (can be NULL)
 * @param closure the closure for the pre-initialization preinit
 *
 * @return the created api in case of success or NULL on error
 *
 * @see afb_api_delete_api
 */
afb_api_t afb_api_new_api(
			afb_api_t api,
			const char *apiname,
			const char *info,
			int noconcurrency,
			int (*preinit)(void*, afb_api_t ),
			void *closure);
```

### afb_api_set_verbs_v2

```C
/**
 * @deprecated use @ref afb_api_set_verbs_v3 instead
 *
 * Set the verbs of the 'api' using description of verbs of the api v2
 *
 * @param api the api that will get the verbs
 * @param verbs the array of verbs to add terminated with an item with name=NULL
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_verb_v2
 * @see afb_api_add_verb
 * @see afb_api_set_verbs_v3
 */
int afb_api_set_verbs_v2(
			afb_api_t api,
			const struct afb_verb_v2 *verbs);
```

### afb_api_set_verbs_v3

```C
/**
 * Set the verbs of the 'api' using description of verbs of the api v2
 *
 * @param api the api that will get the verbs
 * @param verbs the array of verbs to add terminated with an item with name=NULL
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_verb_v3
 * @see afb_api_add_verb
 * @see afb_api_del_verb
 */
int afb_api_set_verbs_v3(
			afb_api_t api,
			const struct afb_verb_v3 *verbs);
```

### afb_api_add_verb

```C
/**
 * Add one verb to the dynamic set of the api
 *
 * @param api the api to change
 * @param verb name of the verb
 * @param info brief description of the verb, can be NULL
 * @param callback callback function implementing the verb
 * @param vcbdata data for the verb callback, available through req
 * @param auth required authorization, can be NULL
 * @param session authorization and session requirements of the verb
 * @param glob is the verb glob name
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_verb_v3
 * @see afb_api_del_verb
 * @see afb_api_set_verbs_v3
 * @see fnmatch for matching names using glob
 */
int afb_api_add_verb(
			afb_api_t api,
			const char *verb,
			const char *info,
			void (*callback)(struct afb_req_x2 *req),
			void *vcbdata,
			const struct afb_auth *auth,
			uint32_t session,
			int glob);
```

### afb_api_del_verb

```C
/**
 * Delete one verb from the dynamic set of the api
 *
 * @param api the api to change
 * @param verb name of the verb to delete
 * @param vcbdata if not NULL will receive the vcbdata of the deleted verb
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afb_api_add_verb
 */
int afb_api_del_verb(
			afb_api_t api,
			const char *verb,
			void **vcbdata);
```

### afb_api_on_event

```C
/**
 * Set the callback 'onevent' to process events in the name of the 'api'.
 *
 * This setting can be done statically using the global variable
 * @ref afbBindingV3.
 *
 * This function replace any previously global event callback set.
 *
 * When an event is received, the callback 'onevent' is called with 3 parameters
 *
 *  - the api that recorded the event handler
 *  - the full name of the event
 *  - the companion JSON object of the event
 *
 * @param api the api that wants to process events
 * @param onevent the callback function that will process events (can be NULL
 *        to remove event callback)
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afbBindingV3
 * @see afb_binding_v3
 * @see afb_api_event_handler_add
 * @see afb_api_event_handler_del
 */
int afb_api_on_event(
			afb_api_t api,
			void (*onevent)(
					afb_api_t api,
					const char *event,
					struct json_object *object));
```

### afb_api_on_init

```C
/**
 * Set the callback 'oninit' to process initialization of the 'api'.
 *
 * This setting can be done statically using the global variable
 * @ref afbBindingV3
 *
 * This function replace any previously initialization callback set.
 *
 * This function is only valid during the pre-initialization stage.
 *
 * The callback initialization function will receive one argument: the api
 * to initialize.
 *
 * @param api the api that wants to process events
 * @param oninit the callback function that initialize the api
 *
 * @return 0 in case of success or -1 on failure with errno set
 *
 * @see afbBindingV3
 * @see afb_binding_v3
 */
int afb_api_on_init(
			afb_api_t api,
			int (*oninit)(afb_api_t api));
```

### afb_api_provide_class

```C
/**
 * Tells that the api provides a class of features. Classes are intended to
 * allow ordering of initializations: apis that provides a given class are
 * initialized before apis requiring it.
 *
 * This function is only valid during the pre-initialization stage.
 *
 * @param api  the api that provides the classes
 * @param name a space separated list of the names of the provided classes
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see afb_api_require_class
 */
int afb_api_provide_class(
			afb_api_t api,
			const char *name);
```

### afb_api_require_class

```C
/**
 * Tells that the api requires a set of class features. Classes are intended to
 * allow ordering of initializations: apis that provides a given class are
 * initialized before apis requiring it.
 *
 * This function is only valid during the pre-initialization stage.
 *
 * @param api  the api that requires the classes
 * @param name a space separated list of the names of the required classes
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see afb_api_provide_class
 */
int afb_api_require_class(
			afb_api_t api,
			const char *name);
```

### afb_api_seal

```C
/**
 * Seal the api. After a call to this function the api can not be modified
 * anymore.
 *
 * @param api the api to be sealed
 */
void afb_api_seal(
			afb_api_t api);
```

### afb_api_delete_api

```C
/**
 * Delete the given api.
 *
 * It is of the responsibility of the caller to don't used the deleted api
 * anymore after this function returned.
 *
 * @param api the api to delete
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see afb_api_new_api
 */
int afb_api_delete_api(
			afb_api_t api);
```

### afb_api_add_alias

```C
/**
 * Create an aliased name 'as_name' for the api 'name'.
 * Calling this function is only allowed within preinit.
 *
 * @param api the api that requires the aliasing
 * @param name the api to alias
 * @param as_name the aliased name of the aliased api
 *
 * @return 0 in case of success or -1 in case of error with errno set appropriately.
 */
int afb_api_add_alias(
			afb_api_t api,
			const char *name,
			const char *as_name);
```


## Legacy functions

The function for legacy calls are still provided for some time because
adaptation of existing code to the new call functions require a small amount
of work.

### afb_api_call_legacy

```C
/**
 * @deprecated try to use @ref afb_api_call instead
 *
 * Calls the 'verb' of the 'apiname' with the arguments 'args' and 'verb'
 * in the name of the binding.
 * The result of the call is delivered to the 'callback' function
 * with the 'callback_closure'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * The 'callback' receives 3 arguments:
 *  1. 'closure' the user defined closure pointer 'closure',
 *  2. 'status' a status being 0 on success or negative when an error occurred,
 *  2. 'result' the resulting data as a JSON object.
 *
 * @param api      The api
 * @param apiname  The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param callback The to call on completion
 * @param closure  The closure to pass to the callback
 *
 * @see also 'afb_api_call'
 * @see also 'afb_api_call_sync'
 * @see also 'afb_api_call_sync_legacy'
 * @see also 'afb_req_subcall'
 * @see also 'afb_req_subcall_sync'
 */
void afb_api_call_legacy(
			afb_api_t api,
			const char *apiname,
			const char *verb,
			struct json_object *args,
			void (*callback)(
					void *closure,
					int status,
					struct json_object *result,
					afb_api_t api),
			void *closure);
```

### afb_api_call_sync_legacy

```C
/**
 * @deprecated try to use @ref afb_api_call_sync instead
 *
 * Calls the 'verb' of the 'apiname' with the arguments 'args' and 'verb'
 * in the name of the binding.
 * 'result' will receive the response.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param api      The api
 * @param apiname  The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param result   Where to store the result - should call json_object_put on it -
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see also 'afb_api_call'
 * @see also 'afb_api_call_sync'
 * @see also 'afb_api_call_legacy'
 * @see also 'afb_req_subcall'
 * @see also 'afb_req_subcall_sync'
 */
int afb_api_call_sync_legacy(
			afb_api_t api,
			const char *apiname,
			const char *verb,
			struct json_object *args,
			struct json_object **result);
```

iv. FUNCTIONS OF CLASS **afb_req**
============================

## General function

### afb_req_is_valid

```C
/**
 * Checks whether the request 'req' is valid or not.
 *
 * @param req the request to check
 *
 * @return 0 if not valid or 1 if valid.
 */
int afb_req_is_valid(
			afb_req_t req);
```

### afb_req_get_api

```C
/**
 * Retrieves the api that serves the request
 *
 * @param req the request whose serving api is queried
 *
 * @return the api serving the request
 */
afb_api_t afb_req_get_api(
			afb_req_t req);
```

### afb_req_get_vcbdata

```C
/**
 * Retrieves the callback data of the verb. This callback data is set
 * when the verb is created.
 *
 * @param req whose verb vcbdata is queried
 *
 * @return the callback data attached to the verb description
 */
void *afb_req_get_vcbdata(
			afb_req_t req);
```

### afb_req_get_called_api

```C
/**
 * Retrieve the name of the called api.
 *
 * @param req the request
 *
 * @return the name of the called api
 *
 * @see afb_api_new_api
 * @see afb_api_add_alias
 */
const char *afb_req_get_called_api(
			afb_req_t req);
```

### afb_req_get_called_verb

```C
/**
 * Retrieve the name of the called verb
 *
 * @param req the request
 *
 * @return the name of the called verb
 */
const char *afb_req_get_called_verb(
			afb_req_t req);
```

### afb_req_addref

```C
/**
 * Increments the count of references of 'req'.
 *
 * @param req the request
 *
 * @return returns the request req
 */
afb_req_t *afb_req_addref(
			afb_req_t req);
```

### afb_req_unref

```C
/**
 * Decrement the count of references of 'req'.
 *
 * @param req the request
 */
void afb_req_unref(
			afb_req_t req);
```


## Logging functions

### afb_req_wants_log_level

```C
/**
 * Is the log message of 'level (as defined for syslog) required for the
 * request 'req'?
 *
 * @param req the request
 * @param level the level to check as defined for syslog:
 *
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @return 0 if not required or a value not null if required
 *
 * @see syslog
 */
int afb_req_wants_log_level(
			afb_req_t req,
			int level);
```

### afb_req_vverbose

```C
/**
 * Send associated to 'req' a message described by 'fmt' and its 'args'
 * to the journal for the verbosity 'level'.
 *
 * 'file', 'line' and 'func' are indicators of position of the code in source files
 * (see macros __FILE__, __LINE__ and __func__).
 *
 * 'level' is defined by syslog standard:
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @param req the request
 * @param level the level of the message
 * @param file the source filename that emits the message or NULL
 * @param line the line number in the source filename that emits the message
 * @param func the name of the function that emits the message or NULL
 * @param fmt the message format as for printf
 * @param args the arguments to the format
 *
 * @see printf
 * @see afb_req_verbose
 */
void afb_req_vverbose(
			afb_req_t req,
			int level, const char *file,
			int line,
			const char * func,
			const char *fmt,
			va_list args);
```

### afb_req_verbose

```C
/**
 * Send associated to 'req' a message described by 'fmt' and following parameters
 * to the journal for the verbosity 'level'.
 *
 * 'file', 'line' and 'func' are indicators of position of the code in source files
 * (see macros __FILE__, __LINE__ and __func__).
 *
 * 'level' is defined by syslog standard:
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 *
 * @param req the request
 * @param level the level of the message
 * @param file the source filename that emits the message or NULL
 * @param line the line number in the source filename that emits the message
 * @param func the name of the function that emits the message or NULL
 * @param fmt the message format as for printf
 * @param ... the arguments of the format 'fmt'
 *
 * @see printf
 */
void afb_req_verbose(
			afb_req_t req,
			int level, const char *file,
			int line,
			const char * func,
			const char *fmt,
			...);
```

## Arguments/parameters function

### afb_req_get

```C
/**
 * Gets from the request 'req' the argument of 'name'.
 *
 * Returns a PLAIN structure of type 'struct afb_arg'.
 *
 * When the argument of 'name' is not found, all fields of result are set to NULL.
 *
 * When the argument of 'name' is found, the fields are filled,
 * in particular, the field 'result.name' is set to 'name'.
 *
 * There is a special name value: the empty string.
 * The argument of name "" is defined only if the request was made using
 * an HTTP POST of Content-Type "application/json". In that case, the
 * argument of name "" receives the value of the body of the HTTP request.
 *
 * @param req the request
 * @param name the name of the argument to get
 *
 * @return a structure describing the retrieved argument for the request
 *
 * @see afb_req_value
 * @see afb_req_path
 */
struct afb_arg afb_req_get(
			afb_req_t req,
			const char *name);
```

### afb_req_value

```C
/**
 * Gets from the request 'req' the string value of the argument of 'name'.
 * Returns NULL if when there is no argument of 'name'.
 * Returns the value of the argument of 'name' otherwise.
 *
 * Shortcut for: afb_req_get(req, name).value
 *
 * @param req the request
 * @param name the name of the argument's value to get
 *
 * @return the value as a string or NULL
 *
 * @see afb_req_get
 * @see afb_req_path
 */
const char *afb_req_value(
			afb_req_t req,
			const char *name);
```

### afb_req_path

```C
/**
 * Gets from the request 'req' the path for file attached to the argument of 'name'.
 * Returns NULL if when there is no argument of 'name' or when there is no file.
 * Returns the path of the argument of 'name' otherwise.
 *
 * Shortcut for: afb_req_get(req, name).path
 *
 * @param req the request
 * @param name the name of the argument's path to get
 *
 * @return the path if any or NULL
 *
 * @see afb_req_get
 * @see afb_req_value
 */
const char *afb_req_path(
			afb_req_t req,
			const char *name);
```

### afb_req_json

```C
/**
 * Gets from the request 'req' the json object hashing the arguments.
 *
 * The returned object must not be released using 'json_object_put'.
 *
 * @param req the request
 *
 * @return the JSON object of the query
 *
 * @see afb_req_get
 * @see afb_req_value
 * @see afb_req_path
 */
struct json_object *afb_req_json(
			afb_req_t req);
```

## Reply functions

The functions **success** and **fail** are still supported.
These functions are now implemented as the following macros:

```C
#define afb_req_success(r,o,i)		afb_req_reply(r,o,NULL,i)
#define afb_req_success_f(r,o,...)	afb_req_reply_f(r,o,NULL,__VA_ARGS__)
#define afb_req_success_v(r,o,f,v)	afb_req_reply_v(r,o,NULL,f,v)
#define afb_req_fail(r,e,i)		afb_req_reply(r,NULL,e,i)
#define afb_req_fail_f(r,e,...)		afb_req_reply_f(r,NULL,e,__VA_ARGS__)
#define afb_req_fail_v(r,e,f,v)		afb_req_reply_v(r,NULL,e,f,v)
```


### afb_req_reply

```C
/**
 * Sends a reply to the request 'req'.
 *
 * The status of the reply is set to 'error' (that must be NULL on success).
 * Its send the object 'obj' (can be NULL) with an
 * informational comment 'info (can also be NULL).
 *
 * For convenience, the function calls 'json_object_put' for 'obj'.
 * Thus, in the case where 'obj' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param req the request
 * @param obj the replied object or NULL
 * @param error the error message if it is a reply error or NULL
 * @param info an informative text or NULL
 *
 * @see afb_req_reply_v
 * @see afb_req_reply_f
 */
void afb_req_reply(
			afb_req_t req,
			struct json_object *obj,
			const char *error,
			const char *info);
```

### afb_req_reply_v

```C
/**
 * Same as 'afb_req_reply_f' but the arguments to the format 'info'
 * are given as a variable argument list instance.
 *
 * For convenience, the function calls 'json_object_put' for 'obj'.
 * Thus, in the case where 'obj' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param req the request
 * @param obj the replied object or NULL
 * @param error the error message if it is a reply error or NULL
 * @param info an informative text containing a format as for vprintf
 * @param args the va_list of arguments to the format as for vprintf
 *
 * @see afb_req_reply
 * @see afb_req_reply_f
 * @see vprintf
 */
void afb_req_reply_v(
			afb_req_t req,
			struct json_object *obj,
			const char *error,
			const char *info,
			va_list args);
```

### afb_req_reply_f

```C
/**
 * Same as 'afb_req_reply' but the 'info' is a formatting
 * string followed by arguments.
 *
 * For convenience, the function calls 'json_object_put' for 'obj'.
 * Thus, in the case where 'obj' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param req the request
 * @param obj the replied object or NULL
 * @param error the error message if it is a reply error or NULL
 * @param info an informative text containing a format as for printf
 * @param ... the arguments to the format as for printf
 *
 * @see afb_req_reply
 * @see afb_req_reply_v
 * @see printf
 */
void afb_req_reply_f(
			afb_req_t req,
			struct json_object *obj,
			const char *error,
			const char *info,
			...);
```

## Subcall functions



### afb_req_subcall

```C
/**
 * Calls the 'verb' of the 'api' with the arguments 'args' and 'verb' in the name of the binding.
 * The result of the call is delivered to the 'callback' function with the 'callback_closure'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * The 'callback' receives 5 arguments:
 *  1. 'closure' the user defined closure pointer 'closure',
 *  2. 'object'  a JSON object returned (can be NULL)
 *  3. 'error'   a string not NULL in case of error
 *  4. 'info'    a string handling some info (can be NULL)
 *  5. 'req'     the req
 *
 * NOTE: For convenience, *json_object_put* is called on 'object' after the
 * callback returns. So, it is wrong to call *json_object_put* in the callback.
 *
 * @param req      The request
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param flags    The bit field of flags for the subcall as defined by @ref afb_req_subcall_flags_t
 * @param callback The to call on completion
 * @param closure  The closure to pass to the callback
 *
 * The flags are any combination of the following values:
 *
 *    - afb_req_x2_subcall_catch_events = 1
 *
 *        the calling API wants to receive the events from subscription
 *
 *    - afb_req_x2_subcall_pass_events = 2
 *
 *        the original request will receive the events from subscription
 *
 *    - afb_req_x2_subcall_on_behalf = 4
 *
 *        the calling API wants to use the credentials of the original request
 *
 *    - afb_req_x2_subcall_api_session = 8
 *
 *        the calling API wants to use its session instead of the one of the
 *        original request
 *
 * @see also 'afb_req_subcall_sync'
 */
void afb_req_subcall(
			afb_req_t req,
			const char *api,
			const char *verb,
			struct json_object *args,
			int flags,
			void (*callback)(
					void *closure,
					struct json_object *object,
					const char *error,
					const char * info,
					afb_req_t req),
			void *closure);
```

### afb_req_subcall_sync

```C
/**
 * Makes a call to the method of name 'api' / 'verb' with the object 'args'.
 * This call is made in the context of the request 'req'.
 * This call is synchronous, it waits untill completion of the request.
 * It returns 0 on success or a negative value on error answer.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * See also:
 *  - 'afb_req_subcall_req' that is convenient to keep request alive automatically.
 *  - 'afb_req_subcall' that doesn't keep request alive automatically.
 *
 * @param req      The request
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param flags    The bit field of flags for the subcall as defined by @ref afb_req_subcall_flags
 * @param object   a pointer where the replied JSON object is stored must be freed using @ref json_object_put (can be NULL)
 * @param error    a pointer where a copy of the replied error is stored must be freed using @ref free (can be NULL)
 * @param info     a pointer where a copy of the replied info is stored must be freed using @ref free (can be NULL)
 *
 * @return 0 in case of success or -1 in case of error
 */
int afb_req_subcall_sync(
			afb_req_t req,
			const char *api,
			const char *verb,
			struct json_object *args,
			int flags,
			struct json_object **object,
			char **error,
			char **info);
```

## Event functions

### afb_req_subscribe

```C
/**
 * Establishes for the client link identified by 'req' a subscription
 * to the 'event'.
 *
 * Establishing subscription MUST be called BEFORE replying to the request.
 *
 * @param req the request
 * @param event the event to subscribe
 *
 * @return 0 in case of successful subscription or -1 in case of error.
 */
int afb_req_subscribe(
			afb_req_t req,
			afb_event_t event);
```

### afb_req_unsubscribe

```C
/**
 * Revokes the subscription established to the 'event' for the client
 * link identified by 'req'.
 * Returns 0 in case of successful subscription or -1 in case of error.
 *
 * Revoking subscription MUST be called BEFORE replying to the request.
 *
 * @param req the request
 * @param event the event to revoke
 *
 * @return 0 in case of successful subscription or -1 in case of error.
 */
int afb_req_unsubscribe(
			afb_req_t req,
			afb_event_t event);
```

## Session functions

### afb_req_context

```C
/**
 * Manage the pointer stored by the binding for the client session of 'req'.
 *
 * If no previous pointer is stored or if 'replace' is not zero, a new value
 * is generated using the function 'create_context' called with the 'closure'.
 * If 'create_context' is NULL the generated value is 'closure'.
 *
 * When a value is created, the function 'free_context' is recorded and will
 * be called (with the created value as argument) to free the created value when
 * it is not more used.
 *
 * This function is atomic: it ensures that 2 threads will not race together.
 *
 * @param req the request
 * @param replace if not zero an existing value is replaced
 * @param create_context the creation function or NULL
 * @param free_context the destroying function or NULL
 * @param closure the closure to the creation function
 *
 * @return the stored value
 */
void *afb_req_context(
			afb_req_t req,
			int replace,
			void *(*create_context)(void *closure),
			void (*free_context)(void*),
			void *closure);
```

### afb_req_context_get

```C
/**
 * Gets the pointer stored by the binding for the session of 'req'.
 * When the binding has not yet recorded a pointer, NULL is returned.
 *
 * Shortcut for: afb_req_context(req, 0, NULL, NULL, NULL)
 *
 * @param req the request
 *
 * @return the previously stored value
 */
void *afb_req_context_get(
			afb_req_t req);
```

### afb_req_context_set

```C
/**
 * Stores for the binding the pointer 'context' to the session of 'req'.
 * The function 'free_context' will be called when the session is closed
 * or if binding stores an other pointer.
 *
 * Shortcut for: afb_req_context(req, 1, NULL, free_context, context)
 *
 *
 * @param req the request
 * @param context the context value to store
 * @param free_context the cleaning function for the stored context (can be NULL)
 */
void afb_req_context_set(
			afb_req_t req,
			void *context,
			void (*free_context)(void*));
```

### afb_req_context_clear

```C
/**
 * Frees the pointer stored by the binding for the session of 'req'
 * and sets it to NULL.
 *
 * Shortcut for: afb_req_context_set(req, NULL, NULL)
 *
 * @param req the request
 */
void afb_req_context_clear(
			afb_req_t req);
```

### afb_req_session_close

```C
/**
 * Closes the session associated with 'req'
 * and delete all associated contexts.
 *
 * @param req the request
 */
void afb_req_session_close(
			afb_req_t req);
```

### afb_req_session_set_LOA

```C
/**
 * Sets the level of assurance of the session of 'req'
 * to 'level'. The effect of this function is subject of
 * security policies.
 *
 * @param req the request
 * @param level of assurance from 0 to 7
 *
 * @return 0 on success or -1 if failed.
 */
int afb_req_session_set_LOA(
			afb_req_t req,
			unsigned level);
```

## Credential/client functions

### afb_req_has_permission

```C
/**
 * Check whether the 'permission' is granted or not to the client
 * identified by 'req'.
 *
 * @param req the request
 * @param permission string to check
 *
 * @return 1 if the permission is granted or 0 otherwise.
 */
int afb_req_has_permission(
			afb_req_t req,
			const char *permission);
```

### afb_req_get_application_id

```C
/**
 * Get the application identifier of the client application for the
 * request 'req'.
 *
 * Returns the application identifier or NULL when the application
 * can not be identified.
 *
 * The returned value if not NULL must be freed by the caller
 *
 * @param req the request
 *
 * @return the string for the application id of the client MUST BE FREED
 */
char *afb_req_get_application_id(
			afb_req_t req);
```

### afb_req_get_uid

```C
/**
 * Get the user identifier (UID) of the client for the
 * request 'req'.
 *
 * @param req the request
 *
 * @return -1 when the application can not be identified or the unix uid.
 *
 */
int afb_req_get_uid(
			afb_req_t req);
```

### afb_req_get_client_info

```C
/**
 * Get informations about the client of the
 * request 'req'.
 *
 * Returns an object with client informations:
 *  {
 *    "pid": int, "uid": int, "gid": int,
 *    "label": string, "id": string, "user": string,
 *    "uuid": string, "LOA": int
 *  }
 *
 * If some of this information can't be computed, the field of the return
 * object will not be set at all.
 *
 * @param req the request
 *
 * @return a JSON object that must be freed using @ref json_object_put
 */
struct json_object *afb_req_get_client_info(
			afb_req_t req);
```

## Legacy functions

### afb_req_subcall_legacy

```C
/**
 * @deprecated use @ref afb_req_subcall
 *
 * Makes a call to the method of name 'api' / 'verb' with the object 'args'.
 * This call is made in the context of the request 'req'.
 * On completion, the function 'callback' is invoked with the
 * 'closure' given at call and two other parameters: 'iserror' and 'result'.
 * 'status' is 0 on success or negative when on an error reply.
 * 'result' is the json object of the reply, you must not call json_object_put
 * on the result.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param req the request
 * @param api the name of the api to call
 * @param verb the name of the verb to call
 * @param args the arguments of the call as a JSON object
 * @param callback the call back that will receive the reply
 * @param closure the closure passed back to the callback
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 */
void afb_req_subcall_legacy(
			afb_req_t req,
			const char *api,
			const char *verb,
			struct json_object *args,
			void (*callback)(
					void *closure,
					int iserror,
					struct json_object *result,
					afb_req_t req),
			void *closure);
```

### afb_req_subcall_sync_legacy

```C
/**
 * @deprecated use @ref afb_req_subcall_sync
 *
 * Makes a call to the method of name 'api' / 'verb' with the object 'args'.
 * This call is made in the context of the request 'req'.
 * This call is synchronous, it waits until completion of the request.
 * It returns 0 on success or a negative value on error answer.
 * The object pointed by 'result' is filled and must be released by the caller
 * after its use by calling 'json_object_put'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param req the request
 * @param api the name of the api to call
 * @param verb the name of the verb to call
 * @param args the arguments of the call as a JSON object
 * @param result the pointer to the JSON object pointer that will receive the result
 *
 * @return 0 on success or a negative value on error answer.
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 */
int afb_req_subcall_sync_legacy(
			afb_req_t req,
			const char *api,
			const char *verb,
			struct json_object *args,
			struct json_object **result);
```

v. FUNCTIONS OF CLASS **afb_event**
==============================

## General functions

### afb_event_is_valid

```C
/**
 * Checks whether the 'event' is valid or not.
 *
 * @param event the event to check
 *
 * @return 0 if not valid or 1 if valid.
 */
int afb_event_is_valid(
			afb_event_t event);
```

### afb_event_name

```C
/**
 * Gets the name associated to 'event'.
 *
 * @param event the event whose name is requested
 *
 * @return the name of the event
 *
 * The returned name can be used until call to 'afb_event_unref'.
 * It shouldn't be freed.
 */
const char *afb_event_name(
			afb_event_t event);
```

### afb_event_unref

```C
/**
 * Decrease the count of references to 'event'.
 * Call this function when the evenid is no more used.
 * It destroys the event_x2 when the reference count falls to zero.
 *
 * @param event the event
 */
void afb_event_unref(
			afb_event_t event);
```

### afb_event_addref

```C
/**
 * Increases the count of references to 'event'
 *
 * @param event the event
 *
 * @return the event
 */
afb_event_t *afb_event_addref(
			afb_event_t event);
```

## Pushing functions

### afb_event_broadcast

```C
/**
 * Broadcasts widely an event of 'event' with the data 'object'.
 * 'object' can be NULL.
 *
 * For convenience, the function calls 'json_object_put' for 'object'.
 * Thus, in the case where 'object' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param event the event to broadcast
 * @param object the companion object to associate to the broadcasted event (can be NULL)
 *
 * @return 0 in case of success or -1 in case of error
 */
int afb_event_broadcast(
			afb_event_t event,
			struct json_object *object);
```

### afb_event_push

```C
/**
 * Pushes an event of 'event' with the data 'object' to its observers.
 * 'object' can be NULL.
 *
 * For convenience, the function calls 'json_object_put' for 'object'.
 * Thus, in the case where 'object' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param event the event to push
 * @param object the companion object to associate to the pushed event (can be NULL)
 *
 * @Return
 *   *  1 if at least one client listen for the event
 *   *  0 if no more client listen for the event
 *   * -1 in case of error (the event can't be delivered)
 */
int afb_event_push(
			afb_event_t event,
			struct json_object *object);
```

vi. FUNCTIONS OF CLASS **afb_daemon**
============================

All the functions of the class **afb_daemon** use the default api.
This are internally aliased to the corresponding function of class afb_api_t.

```C
/**
 * Retrieves the common systemd's event loop of AFB
 */
struct sd_event *afb_daemon_get_event_loop();

/**
 * Retrieves the common systemd's user/session d-bus of AFB
 */
struct sd_bus *afb_daemon_get_user_bus();

/**
 * Retrieves the common systemd's system d-bus of AFB
 */
struct sd_bus *afb_daemon_get_system_bus();

/**
 * Broadcasts widely the event of 'name' with the data 'object'.
 * 'object' can be NULL.
 *
 * For convenience, the function calls 'json_object_put' for 'object'.
 * Thus, in the case where 'object' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * Calling this function is only forbidden during preinit.
 *
 * Returns the count of clients that received the event.
 */
int afb_daemon_broadcast_event(
			const char *name,
			struct json_object *object);

/**
 * Creates an event of 'name' and returns it.
 *
 * Calling this function is only forbidden during preinit.
 *
 * See afb_event_is_valid to check if there is an error.
 */
afb_event_t afb_daemon_make_event(
			const char *name);

/**
 * @deprecated use bindings version 3
 *
 * Send a message described by 'fmt' and following parameters
 * to the journal for the verbosity 'level'.
 *
 * 'file', 'line' and 'func' are indicators of position of the code in source files
 * (see macros __FILE__, __LINE__ and __func__).
 *
 * 'level' is defined by syslog standard:
 *      EMERGENCY         0        System is unusable
 *      ALERT             1        Action must be taken immediately
 *      CRITICAL          2        Critical conditions
 *      ERROR             3        Error conditions
 *      WARNING           4        Warning conditions
 *      NOTICE            5        Normal but significant condition
 *      INFO              6        Informational
 *      DEBUG             7        Debug-level messages
 */
void afb_daemon_verbose(
			int level,
			const char *file,
			int line,
			const char * func,
			const char *fmt,
			...);

/**
 * @deprecated use bindings version 3
 *
 * Get the root directory file descriptor. This file descriptor can
 * be used with functions 'openat', 'fstatat', ...
 *
 * Returns the file descriptor or -1 in case of error.
 */
int afb_daemon_rootdir_get_fd();

/**
 * Opens 'filename' within the root directory with 'flags' (see function openat)
 * using the 'locale' definition (example: "jp,en-US") that can be NULL.
 *
 * Returns the file descriptor or -1 in case of error.
 */
int afb_daemon_rootdir_open_locale(
			const char *filename,
			int flags,
			const char *locale);

/**
 * Queue the job defined by 'callback' and 'argument' for being executed asynchronously
 * in this thread (later) or in an other thread.
 * If 'group' is not NUL, the jobs queued with a same value (as the pointer value 'group')
 * are executed in sequence in the order of there submission.
 * If 'timeout' is not 0, it represent the maximum execution time for the job in seconds.
 * At first, the job is called with 0 as signum and the given argument.
 * The job is executed with the monitoring of its time and some signals like SIGSEGV and
 * SIGFPE. When a such signal is catched, the job is terminated and reexecuted but with
 * signum being the signal number (SIGALRM when timeout expired).
 *
 * Returns 0 in case of success or -1 in case of error.
 */
int afb_daemon_queue_job(
			void (*callback)(int signum, void *arg),
			void *argument,
			void *group,
			int timeout);

/**
 * Tells that it requires the API of "name" to exist
 * and if 'initialized' is not null to be initialized.
 * Calling this function is only allowed within init.
 *
 * Returns 0 in case of success or -1 in case of error.
 */
int afb_daemon_require_api(
			const char *name,
			int initialized);

/**
 * Create an aliased name 'as_name' for the api 'name'.
 * Calling this function is only allowed within preinit.
 *
 * Returns 0 in case of success or -1 in case of error.
 */
int afb_daemon_add_alias(const char *name, const char *as_name);

/**
 * Creates a new api of name 'api' with brief 'info'.
 *
 * Returns 0 in case of success or -1 in case of error.
 */
int afb_daemon_new_api(
			const char *api,
			const char *info,
			int noconcurrency,
			int (*preinit)(void*, struct afb_api_x3 *),
			void *closure);
```

vii. FUNCTIONS OF CLASS **afb_service**
==============================

All the functions of the class **afb_service** use the default api.

All these function are deprecated, try to use functions of class **afb_api** instead.

## afb_service_call

```C
/**
 * @deprecated try to use @ref afb_api_call instead
 *
 * Calls the 'verb' of the 'api' with the arguments 'args' and 'verb' in the name of the binding.
 * The result of the call is delivered to the 'callback' function with the 'callback_closure'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * The 'callback' receives 5 arguments:
 *  1. 'closure' the user defined closure pointer 'closure',
 *  2. 'object'  a JSON object returned (can be NULL)
 *  3. 'error'   a string not NULL in case of error but NULL on success
 *  4. 'info'    a string handling some info (can be NULL)
 *  5. 'api'     the api
 *
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param callback The to call on completion
 * @param closure  The closure to pass to the callback
 *
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 * @see afb_api_call_sync
 */
void afb_service_call(
			const char *api,
			const char *verb,
			struct json_object *args,
			void (*callback)(
					void *closure,
					struct json_object *object,
					const char *error,
					const char * info,
					afb_api_t api),
			void *closure);
```

## afb_service_call_sync

```C
/**
 * @deprecated try to use @ref afb_api_call_sync instead
 *
 * Calls the 'verb' of the 'api' with the arguments 'args' and 'verb' in the name of the binding.
 * 'result' will receive the response.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param object   Where to store the returned object - should call json_object_put on it - can be NULL
 * @param error    Where to store the copied returned error - should call free on it - can be NULL
 * @param info     Where to store the copied returned info - should call free on it - can be NULL
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see afb_req_subcall
 * @see afb_req_subcall_sync
 * @see afb_api_call
 */
int afb_service_call_sync(
			const char *api,
			const char *verb,
			struct json_object *args,
			struct json_object **object,
			char **error,
			char **info);
```

## afb_service_call_legacy

```C
/**
 * @deprecated try to use @ref afb_api_call instead
 *
 * Calls the 'verb' of the 'api' with the arguments 'args' and 'verb'
 * in the name of the binding.
 * The result of the call is delivered to the 'callback' function with
 * the 'callback_closure'.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * The 'callback' receives 3 arguments:
 *  1. 'closure' the user defined closure pointer 'closure',
 *  2. 'status' a status being 0 on success or negative when an error occurred,
 *  2. 'result' the resulting data as a JSON object.
 *
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param callback The to call on completion
 * @param closure  The closure to pass to the callback
 *
 * @see also 'afb_api_call'
 * @see also 'afb_api_call_sync'
 * @see also 'afb_api_call_sync_legacy'
 * @see also 'afb_req_subcall'
 * @see also 'afb_req_subcall_sync'
 */
void afb_service_call_legacy(
			const char *api,
			const char *verb,
			struct json_object *args,
			void (*callback)(
					void *closure,
					int status,
					struct json_object *result,
					afb_api_t api),
			void *closure);
```

## afb_service_call_sync_legacy

```C
/**
 * @deprecated try to use @ref afb_api_call_sync instead
 *
 * Calls the 'verb' of the 'api' with the arguments 'args' and 'verb' in the
 * name of the binding. 'result' will receive the response.
 *
 * For convenience, the function calls 'json_object_put' for 'args'.
 * Thus, in the case where 'args' should remain available after
 * the function returns, the function 'json_object_get' shall be used.
 *
 * @param api      The api name of the method to call
 * @param verb     The verb name of the method to call
 * @param args     The arguments to pass to the method
 * @param result   Where to store the result - should call json_object_put on it -
 *
 * @returns 0 in case of success or a negative value in case of error.
 *
 * @see also 'afb_api_call'
 * @see also 'afb_api_call_sync'
 * @see also 'afb_api_call_legacy'
 * @see also 'afb_req_subcall'
 * @see also 'afb_req_subcall_sync'
 */
int afb_service_call_sync_legacy(
			const char *api,
			const char *verb,
			struct json_object *args,
			struct json_object **result);
```