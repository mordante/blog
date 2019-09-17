---
layout: post
title:  "String_view support for regex – what does it do?"
date:   2019-09-17
categories: C++ string_view regex standardization
tags: C++ string_view regex standardization
---
This is the second part about posts what my "string_view support for regex"
proposal entails. This part explains  what it does for the user.
[part 1][part1] explains why I started to work on it,

If my [draft] proposal will be accepted in C++23 it will make using
`std::string_view` with `std::regex` much more pleasant experience. This post
shows the most promintent features for the user's point of view.

### TL;DR ###

It will be possible to use a `std::svmatch` instead of a `std::smatch` to get
the match results when using a `std::string_view` in `std::regex_match` or
`std::regex_search`. The match results and its sub matches can return a
`std::string_view`.

### Regex ###

The `std::basic_regex` will get the following overloads taking a `std::string_view`:
  - constructor
  - `operator=`
  - member function `assign`

### Sub_match ###

The `std::sub_match` will get two new typedefs for
`std::string_view::const_iterator` and `std::wstring_view::const_iterator`. They
will not often be used directly by the user, but are added for completeness.

The `std::sub_match` class will get the following additional member functions:
  - `operator string_view_type() const;`
  - `string_view_type view() const;`
  - `int compare(string_view_type sv) const;`

This allows implicit conversion to a `std::string_view`. Whether this will
be accepted is not certain: `std::sub_match` can now implitely convert to both
`std::string` and to `std::string_view`.

The new functions in this class make it easier to get a `std::string_view` of
the match and do not require a `std::string` to look at the result.


### Match_results ###

The `std::match_result` will get two new typedefs for `svmatch` and `wvsmatch`.
These will be often used in user code!

The `std::match_result` class will get the following additional member
functions:
  - `string_view_type view(size_type sub = 0) const;`
  - Two overloads of `format` to use a `std::string_view` as format argument.

The new typdefs make it easy to use a `std::match_result` for a
`std::string_view` the `view()` function makes it easier to get a
`std::string_view` of the match and do not require a `std::string` to look at
the result.


### Regex_match & regex_search ###

The `std::regex_match` and `std::regex_search` functions get overloads to take a
`std::string_view` as input.


### regex_replace ###

The `std::regex_replace` function get overloads to take a `std::string_view` as
input and format argument.


### Hidden goodies ###

There are some more hidden goodies in the proposal. Most constructions of
temporary `std::string` objects will become temporary `std::string_view`
objects. The latter is much cheaper to create and consumes less memory. There
are several other typedefs using a `std::string_view`, but will probably not be
used a lot.


### Caveat emptor ###

Another tiny issue with `std::sub_match` is the fact it takes `BidirectionalIterator`
which does not need to be continious. For most code the `BidirectionalIterator`
will also be continuous, for example:
  - `const char *`
  - `std::string`
  - `std::string_view`
  - `std::vector`
  - `std::array`

But the following types are not continuos:
  - `std::deque`
  - `std::list`

In these cases the `std::string_view` operator will not be available and the
`view()` member returns the same as `str()`.

The latest [draft] of the proposal contains all details. Hopefully this will
become part of C++23.


[part1]: {{site.baseurl}}{% post_url 2019-09-17-string_view_support_for_regex-part_1 %}
[draft]: {{site.baseurl}}/downloads/Dxxxx_string_view_support_for_regex.pdf
