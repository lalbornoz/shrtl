## Bourne shell coding rules

> N.B. Input sanitisation is mandatory whenever input may form part of a parameter name, most usually
when indexing with input as a key into a (pseudo-)hash, e.g. PKG_ZSH_<...input...>; failing to do so
may introduce security vulnerabilities (e.g.: $(arbitrary_command) and ${arbitrary_variable} facilitating
code execution and information disclosure, resp.)  
Do not use this code and these coding rules if this is not possible or impractical.

*(reproduced from &lbrack;[midipix_build](https://github.com/lalbornoz/midipix_build/blob/main/etc/README.md)&rbrack;)*

If no rationale is specified for any specific point, the rationale is avoidance of undefined behaviour
and/or implicit behaviour contingent on often subtle special cases, both of which are prone to cause
hard to debug or even diagnose bugs.

### Shell options

1)  The `noglob` option is set at all times *except* for when globbing is required, e.g.:
    `set +o noglob; files=*; set -o noglob`
2)  The `nounset` option is set at all times; if a parameter is to be expanded that may be
    unset, use the `${parameter:-word}` expansion format.
3)  The `errexit` option is unset at all times *except* for top-level subshell code that does
    *not* engage in conditional evaluation, e.g. `([...] set -o errexit; [...]) &` due to its
    implicit unsetting when a/any function is subject to conditional evaluation, e.g.: `[...]
    set -o errexit; [...] if some_function; then [ ... ] fi`

### Quoting and brace-enclosing

4)  Quoting with single quotes or double quotes is mandatory at all times *except* for when field 
    splitting subsequent to parameter expansion, etc. is required, e.g.: `params="a b c";
    param="single parameter"; fn ${params} "${param}"`
5)  Enclosing parameter names in braces is mandatory at all times, e.g.: `${parameter_name123}`

### Checking parameter status

6)  Checking for whether a parameter is set/non-empty or unset/empty is expressed with the following
    the idiom: `[ "${parameter:+1}" = 1 ]` and `[ "${parameter:+1}" != 1 ]` 
  
    This is necessary to avoid the potentially costly expansion of a parameter's value when only its
    status of being set/non-empty or unset/empty is of concern, particularly when this is done repeatedly.

### Functions

7)  Functions must explicitly return either `0` or `1` at all times *except* for when specific
    return exit statuses are specified according to the function's interface contract.
8)  Function names must have namespace prefixes followed by a `p` in the topmost namespace prefix
    field in private functions, e.g.: `rtlp_log_do_something() { [...] }`
9)  The non-POSIX but extremely widely available `local` command must be used at at all times
    in functions on all function-local variables.  
  
    This is necessary in order to avoid polluting the (global) parameter namespace.
10) Function-local variables must be lower case and prefixed with a `_`, a prefix comprised of the
    initial letters of each word in the function's name plus, in private functions, the `p` topmost
    namespace prefix field postfix, and a `_`, where words are separated by `_` characters, e.g.:
    `rtlp_log_do_something() { local _rplds_parameter=""; [...] }` and `ex_something() {
    local _es_parameter=""; [ ...] }` 
  
    This is necessary in order to prevent implicit conflicts between two or more functions that share
    a call path and would otherwise use the same variables names, such as `function1() { local
    list="${1}"; function2 "${list}"; }; function2() { local list="${2}"; }`, particularly when those
    functions acess and/or mutate each other's variables. Additionally, local variables are thus marked
    with a `_` prefix and by being lower case.

### `eval` rules

11) The following special characters must be escaped with a backslash at all times in `eval`
    expressions: `<newline>`, `"`, `'`, `[`, `\`, `]`, `<backtick>`, `$`
12) If parameter expansion is to be evaluated with respect to a parameter reference, the following idiom
    must be used at all times: `eval [...]\${${rparameter[...]}}`
13) If quoting with single quotes or double quotes is required - see 4) - or brace-enclosing - see 5) -
    and with respect to 11), viz. the single quotes or double quotes have been escaped with a backslash,
    something must be evaluated as a single field, then that field must be doubly quoted with single or
    double quotes, where the second set of quotes must not be escaped with a backslash, e.g.: `
    rparameter=parameter; eval ${rparameter}="\"a b c\""`. 
    This is very rarely required.

### Passing arguments

14) The `fork/exec/write`-`read` pattern where a parameter is set to the result of a command
    substitution expression which is executed in a subshell process, e.g.: `parameter="$(function)"`
    is prohibited at all times *except* when an actual external command is invoked, e.g. `sort(1)`
    or `sed(1)`.  
  
    This is necessary due to the very significant cost of the mentioned pattern, particularly concerning
    primitive e.g. list or string processing functions with high contention.
15) If a function is to return, produce, evaluate to, etc. an arbitrary value apart from the exit status,
    it shall do so by receiving references to the target variables in the callers' scope from the caller
    prefixed with a single `$` character, e.g. `function \$parameter` and `function() {
    local _fn_rparameter="${1#\$}"; [..]; }` which are then set with `eval` expressions: `
    function() { local _fn_rparameter="${1#\$}"; [...]; eval ${_fn_rparameter}=\'a b c\'; };` Refer to
    11)-13) for the rules concerning `eval`.  

    This is necessary due to 14) as well as the absence of any other calling convention other than using
    implicit global variables, e.g.: `function1() { VARIABLE=; function2; }; function2() { VARIABLE=1;
    }`
  
[modeline]: # ( vim: set tw=0: )
