---
layout: post
title: "A simple Bash template"
date: 2018-08-21 08:59:23
categories: source code
tags: bash
---

Shell scripting is a simple and powerful way of performing complex tasks.

After being a lot of years implementing them, I have learned that the most important thing is simplifying them instead of adding complexity, as it happens with many things in life.

So, my simple template contains:

* A main function.
* A function to to print how to use it.
* A way to get user parameters (both mandatory and optional). I like getopts.
* A simple logger. I like nanosecond precision.
* A function to clean up things before exit. E.g.: remove a lock or kill some process.

And don't forget that you have tools to evaluate your scripts, like the great [ShellCheck](https://github.com/koalaman/shellcheck).

{% highlight bash %}
#!/bin/bash

SCRIPT_NAME=$(basename "$0")
DEFAULT_OPTION=0

trap clean EXIT

usage ()
{
    echo -e "Usage: $SCRIPT_NAME -v VALUE [OPTIONS]\n\
Write script description.\n\
Options:\n\
\t-o OPTION\tWrite option description. Default: $DEFAULT_OPTION\n"
}

log ()
{
    now=$(date +"%Y-%m-%d %T.%N")
    echo "[$now] $1"
}

clean ()
{
    log "Exiting..."

    #TODO

    log "Bye"
}

main ()
{
    value=""
    option=$DEFAULT_OPTION

    while getopts v:o: OPTNAME
    do
        case "$OPTNAME" in
            v)
                value=$OPTARG
                ;;
            o)
                option=$OPTARG
                ;;
            *)
                usage
                exit 1
                ;;
        esac
    done

    # check mandatory parameters
    if [ x"$value" == x ]
    then
        usage
        exit 1
    fi

    #TODO
    log "Running..."

    exit 0
}

main "$*"
exit $?
{% endhighlight %}
