---
layout: post
title:  "String_view support for regex – why do we need it?"
date:   2019-09-17
categories: C++ string_view regex standardization
tags: C++ string_view regex standardization
---
This is the first part about posts what my "string_view support for regex"
proposal entails. This part explains why I started to work on it,
[part 2][part2] explains what it does for the user.

While working on toy project I wanted to parse the command line input with
several simple regexes. Using C++17 I thought it would be nice to 'copy' the
input using a `std::string_view`.

{% highlight c++ lineos %}
void foo(std::string_view input) {
	std::regex re{"foo.*bar"};
	std::string i{input};
	std::smatch match;
	if (regex_match(i, match, re))
		std::cout << "Found " << match[0] << "\n";
}
{% endhighlight %}

It works, but obviously I was not happy with it. I used a `std::string_view` to
avoid copying and as a result I needed to make a copy to use it in the regex.
Using `const std::string&` for `input` would have been more efficient.

Luckily it is possible to use a `std::string_view` as input if you use
iterators.

{% highlight c++ lineos %}
void foo(std::string_view input) {
	std::regex re{"foo.*bar"};
	std::match_results<std::string_view::const_iterator> match;
	if (regex_match(input.begin(), input.end(), match, re))
		std::cout << "Found " << match[0] << "\n";
}
{% endhighlight %}

It works, but the call to `std::cout` still copies the matched data to a
`std::string`. But no problem that can also be solved. Since the code uses
uniform initialization we need a `static_cast` since `length()` returns a
`difference_type`. This type differs from `std::string_view::size_type`
and narrowing is not allowed.

{% highlight c++ lineos %}
void foo(std::string_view input) {
	std::regex re{"foo.*bar"};
	std::match_results<std::string_view::const_iterator> match;
	if (regex_match(input.begin(), input.end(), match, re)) {
		std::string_view output{&*match[0].first,
		                        static_cast<size_t>(match[0].length())};
		std::cout << "Found " << output << "\n";
	}
}
{% endhighlight %}

So that will be the best we can do, hopefully in C++23 the following code will
be as efficient at the previous example.

{% highlight c++ lineos %}
void foo(std::string_view input) {
	std::regex re{"foo.*bar"};
	std::svmatch match;
	if (regex_match(input, match, re))
		std::cout << "Found " << match[0] << "\n";
}
{% endhighlight %}

That looks a lot like the initial example, but the `match` is a `svmatch`, short
for string_view match. The `std::operator<<` will automatically use the new
`view()` member function of the `std::sub_match`.

I'm working on a paper which I hope will make it into C++23. The latest [draft]
of the proposal is available. [Part 2][part2] will go into more details what
this proposal does.


[part2]: {{site.baseurl}}{% post_url 2019-09-17-string_view_support_for_regex-part_2 %}
[draft]: {{site.baseurl}}/downloads/Dxxxx_string_view_support_for_regex.pdf
