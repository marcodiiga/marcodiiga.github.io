---
layout: post
title:  "Enhancing Pygments lexer for C++ code"
---

[Pygments](http://pygments.org/) is a python based syntax highlighter for a large variety
of languages and it is also used by default on Jekyll generators.

Readability of the code is a subjective matter but one small improvement that
I would have loved is being able to modify the style for C++ include directives.
By default Pygments treats preprocessor directives as a subset of comments and
does not distinguish the `#include` directive from the include path

![include first](/images/posts/enhancingpygments1.png)

To get pygments to correctly add a different style for a custom token or a specific
code portion (assuming it can be identified in the lexing phase) there are two
necessary steps

1) Add the custom token (if needed) to the `vendor/pygments-main/pygments/token.py` file

{% highlight python %}
# Map standard token types to short names, used in CSS class naming.
# If you add a new item, please be sure to run this file to perform
# a consistency check for duplicate values.
STANDARD_TYPES = {
    Token:                         '',

    Text:                          '',
    Whitespace:                    'w',
    Escape:                        'esc',
    Error:                         'err',
    Other:                         'x',
    ...

    Comment:                       'c',
    Comment.Multiline:             'cm',
    Comment.Preproc:               'cp',
    Comment.PreprocFile:           'cpf', <--
    Comment.Single:                'c1',
    Comment.Special:               'cs',
    ...
{% endhighlight %}

2) Instruct the language lexer (in this case the **CppLexer** contained in `lexers/compiled.py`)
to perform a different lexing when handling include directives

{% highlight python %}
class CFamilyLexer(RegexLexer):
    """
    For C family source code.  This is used as a base class to avoid repetitious
    definitions.
    """

    #: optional Comment or Whitespace
    _ws = r'(?:\s|//.*?\n|/[*].*?[*]/)+'
    #: only one /* */ style comment
    _ws1 = r'\s*(?:/[*].*?[*]/\s*)*'

    tokens = {
        'whitespace': [
            # preprocessor directives: without whitespace
            ('^#if\s+0', Comment.Preproc, 'if0'),
            ('^#', Comment.Preproc, 'macro'),       <--
	...
	'macro': [
            (r'(include)(' + _ws1 + ')([^\n]+)',  <--
                      bygroups(Comment.Preproc, Text, Comment.PreprocFile)),
            (r'[^/\n]+', Comment.Preproc),
	...
{% endhighlight %}

This gets Pygments to do the right thing when parsing C++ include directives.

![include after](/images/posts/enhancingpygments2.png)

However it has to be noted that this only works when an 'enhanced' version of
Pygments is used to generate a static site.
