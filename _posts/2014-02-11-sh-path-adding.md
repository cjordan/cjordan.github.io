---
layout: post
title:  "Handy zsh/bash functions for version-numbered paths"
date:   2014-02-11
---

There's a few pieces of software around the place that can be painful to include in your \\$PATH when they update. Examples include Ruby gems (\\$HOME/.gem/ruby/**#version#**/bin) and CASA (/usr/local/casapy-**#version#**/bin). I'm lazy; I don't want to update my zshrc every time I have a new version of some software.

I believe I've finally solved this issue:

{% highlight bash %}
append_path () {
    [[ -d $1 && $PATH != *$1* ]] && PATH+=:$1
}
each_version () {
    setopt nullglob
    if stat --printf='' $1* 2>/dev/null; then
        for x in $1*; do
            temp_path=($x $temp_path)
        done
        for x in $temp_path; do
            append_path $x/$2
        done
    fi
    unsetopt nullglob
}
{% endhighlight %}

**Note that `setopt` and `unsetopt` are a zsh-isms. Use `shopt -s` and `shopt -u` for bash.**

Put these in your .zshrc (or .bashrc), then use them as:

{% highlight bash %}
append_path ~/bin

# Ruby gems
each_version ~/.gem/ruby/ bin

# CASA
each_version /usr/local/casapy bin
{% endhighlight %}

`append_path` is straight-forward; add this folder to the path if it isn't there already. It's nice because it will test if a directory exists before attempting to add it - this way, your rc is portable.

`each_version` is a little more tricky. The trailing slash means "I don't care what the version number is in here, just add the 'bin' directory it contains." No trailing slash uses the root of the directory name to search; in the above case, 'casapy'.
