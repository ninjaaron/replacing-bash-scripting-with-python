Replacing Bash Scripting with Python
====================================

If I didn't cover something you want to know about or you find another
problem, please open an issue on github_!

.. _github:
  https://github.com/ninjaaron/replacing-bash-scripting-with-python

.. contents::

Introduction
------------
The Unix shell is one of my favorite inventions ever. It's genius, plain
and simple. The idea is that the user environment is a Turing-complete,
declarative programming language. It has a dead-simple model for dealing
with I/O and concurrency, which are notoriously difficult in most other
languages.

For problems where the data can be expressed as a stream of similar
objects separated by newlines to be processed concurrently through a
series of filters and handles a lot of I/O, it's difficult to think of a
more ideal language than the shell. A lot of the core parts on a Unix or
Linux system are designed to express data in such formats.

If the Shell is so great, what's the problem?
+++++++++++++++++++++++++++++++++++++++++++++
The problem is if you want to do basically anything else, e.g. write
logic, use control structures, handle complex data... You're going
to have big problems. When Bash is coordinating external programs, it's
fantastic. When it's doing any work whatsoever itself, it disintegrates
into a pile of garbage.

For me, the fundamental problem with Bash and other shell dialects is
that text is identifiers and identifiers are text -- and basically
everything else is also text. In some sense, this makes the shell a
homoiconic language, which theoretically means it might have an
interesting metaprogramming story, until you realize that it basically
just amounts to running ``eval`` on strings, which is a feature in
basically any interpreted language today, and one that is frequently
considered harmful. The problem with ``eval`` is that it's a pretty
direct path to arbitrary code execution. This is great if arbitrary code
execution is actually what you're trying to accomplish (like, say, in an
HTML template engine), but it's not generally what you want.

Bash basically defaults to evaling everything. This very handy for
interactive use, since it cuts down in the need for a lot of explicit
syntax when all you really want to do is, say, open a file in a text
editor. This is pretty darn bad in a scripting context because it turns
the entire language into an injection honeypot. Yes, it is possible and
not so difficult to write safe Bash once you know the tricks, but it
takes extra consideration and it is easy to forget or be lazy about it.
Writing three or four lines of safe Bash is easy; two-hundred is quite a
bit more challenging.

Bash has other problems. The syntax that isn't native to the Bourne
Shell feels really ugly and bolted-on. For example, most modern shells
have arrays. Let's look at the syntax for iterating on an array, but
let's take the long way there.

.. code:: bash

  $ foo='this   and   that' # variable asignment
  $ echo $foo
  this and that
  $ # Oh dear. Text inside the variable was split into arguments on
  $ # whitespace, because eval all the things.
  $
  $ # To avoid this insane behavior, do the obvious thing: use string
  $ # interpolation. :-(
  $ echo "$foo"
  this   and   that

What does this have to do with iterating on arrays? Unfortunately, the
answer is "something."

To properly iterate on the strings inside of an array (the only thing
which an array can possibly contain), you also use variable
interpolation syntax.

.. code:: bash

  for item in "${my_array[@]}"; do
      stuff with "$item"
  done

Why would string interpolation syntax ever be used to iterate over items
in an array? I have some theories, but they are only that. I could tell
you, but it wouldn't make this syntax any less awful. If you're not too
familiar with Bash, you may also (rightly) wonder what this ``@`` is, or
why everything is in curly braces.

The answer to all these questions is more or less that they didn't want
to do anything that would break compatibility with ancient Unix shell
scripts, which didn't have these features. Everything just got
shoe-horned in with the weirdest syntax you can imagine. Bash actually
has a lot of features of modern programming languages, but the problem
is that the syntax provided to access them is completely contrary to
logic and dictated by legacy concerns.

The Bash IRC channel has a very helpful bot, greybot, written by one of
the more important Bash community members and experts, greycat. This bot
is written in Perl. I once asked why it wasn't written in Bash, and only
got one answer: "greycat wanted to remain sane."

And really, that answer should be enough. Do you want to remain sane? Do
you want people who maintain your code in the future not to curse your
name? Don't use Bash. Do your part in the battle against mental illness.

Why Python?
+++++++++++
No particular reason. Perl and Ruby are also flexible, easy-to-write
languages that have robust support for administrative scripting and
automation. I would recommend against Perl for beginners because it has
some similar issues to Bash: it was a much smaller language when it was
created, and a lot of the syntax for the newer features has a bolted-on
feeling. However, if one knows Perl well and is comfortable with it,
it's well suited to the task, and is still a much saner choice for
non-trivial automation scripts.

Node.js is also starting to be used for administrative stuff these days,
so that could also be an option. I've been investigating the possibility
of using Julia for this as well. Anyway, most interpreted languages seem
to have pretty good support for this kind of thing, and you should just
choose one that you like and is widely available on Linux and other
\*nix operating systems.

They main reason I would recommend Python is if you already know it. If
you don't know anything besides BASH (or BASH and lower-level languages
like C or even Java), Python is a reasonable choice for your next
language. It has a lot of mature, fast third-party libraries in a lot of
domains -- science, math, web, machine learning, etc. It's also
generally considered easy to learn and has become a major teaching
language.

The other very compelling reason to learn Python is that it is the
language covered in this very compelling tutorial.

Learn Python
++++++++++++
This tutorial isn't going to teach you the Python core language, though
a few built-in features will be covered. If you need to learn it, I
highly recommend the `official tutorial`_, at least through chapter 5.
Through chapter 9 would be even better, and you might as well just read
the whole thing at that point.

If you're new to programming, you might try the book *Introducing
Python* or perhaps *Think Python*. You may see a lot of recommendations
for *Learn Python the Hard Way*. I think this method is flawed, though I
do appreciate that it was written by someone with strong opinions about
correctness, which has some benefits.

This tutorial assumes Python 3.5 or higher, though it may sometimes use
idioms from 3.6, and I will attempt to document when have used an idiom
which doesn't work in 3.4, which is apparently the version that ships
with the latest CentOS and SLES. Use 3.6 if you can. It has some cool
new features, but the implementation of dictionaries (Python's hash map)
was also overhauled in this version of Python, which sort of undergirds
the way the whole object system is implemented and therefore is a major
win all around.

Basically, always try to use whatever the latest version of Python is.
Do not use Python 2. It will be officially retired in 2020. That's two
years. If a library hasn't been ported to Python 3 yet, it's already
dead, just that its maintainers might not know it yet.

One last note about this tutorial: It doesn't explain *so much*. I have
no desire to rewrite things that are already in the official
documentation. It frequently just points to the relevant documentation
for those wishing to do the kinds of tasks that Bash scripting is
commonly used for.

.. _official tutorial: https://docs.python.org/3/tutorial/index.html

Reading and Writing Files
-------------------------
If you're going to do any kind of administration or automation on a Unix
system, the idea of working with files is pretty central. The great
coreutils like ``grep``, ``sed``, ``awk``, ``tr``, ``sort``, etc., they
are all designed to go over text files line by line and do... something
with the content of that line. Any shell scripter knows that these
"files" aren't always really files. Often as not, it's really dealing
with the output of another process and not a file at all. Whatever the
source, the organizing principle is streams of text divided by newline
characters. In Python, this is what we'd call a "file-like object."

Because the idea of working with text streams is so central to Unix
programming, we start this tutorial with the basics of working with text
files and will go from there to other streams you might want to work
with.

One handy thing in the shell is that you never really need file handles.
The file name is typically all you have to type to loop over lines in a
file would be something like:

.. code:: Bash

  while read line; do
      stuff with "$line"
  done < my_file.txt

(Don't use this code. You actually have to do some things with $IFS to
make it safe. Don't use any of my Bash examples. Don't use Bash! The
proper one is ``while IFS= read -r line``, but that just raises more
questions.)

In Python, you need to turn a path into a file object. The above loop
would be something like this:

.. code:: Python

  with open('my_file.txt') as my_file:
      for line in my_file:
          do_stuff_with(line.rstrip())

  ## the .rstrip() method is optional. It removes trailing whitespace
  ## from the line (including the newline character).

Let's take that apart.

The ``open()`` function returns a file object. If you just send it the
path name as a string, it's going to assume it's a text file in the
default system encoding (UTF-8, right?), and it is opened only for
reading. You can, of course, do ``my_file = open('my_file.txt')`` as
well. When you use ``with x as y:`` instead of asignment, it ensures the
object is properly cleaned up when the block is exited using something
called a "context manager". You can do ``my_file.close()`` manually, but
the ``with`` block will ensure that happens even you hit an error
without having to write a lot of extra code.

The gross thing about context managers is that that they add an extra
level of indentation. Here's a helper function you can use to open a
context manager for something you want cleaned up after you loop.

.. code:: Python

  def iter_with(obj):
      with obj:
          yield from obj

and then you use it like this:

.. code:: Python

  for line in iter_with(open('my_file.txt')):
      do_stuff_with(line)

``yield from`` means it's a `generator function`_, and it's
handing over control to a sub-iterator (the file object, in this case)
until that iterator runs out of things to return. Don't worry if that
doesn't make sense. It's a more advanced Python topic and not necessary
for administrative scripting.

If you don't want to iterate on lines, which is the most
memory-efficient way to deal with text files, you can slurp entire
contents of a file at once like this:

.. code:: Python

  with open('my_file.txt') as my_file:
      file_text = my_file.read()
      ## or
      lines = list(my_file)
      ## or with newline characters removed
      lines = my_file.read().splitlines()

  ## This code wouldn't actually run because the file hasn't been
  ## rewound to the beginning after it's been read through.

  ## Also note: list(my_file). Any function that takes an iterable can
  ## take a file object.



You can also open files for writing with, like this:

.. code:: Python

  with open('my_file.txt', 'w') as my_file:
      my_file.write('some text\n')
      my_file.writelines(['a\n', 'b\n', 'c\n'])
      print('another line', file=my_file)        # print adds a newline.


The second argument of ``open()`` is the *mode*. The default mode is
``'r'``, which opens the file for reading text. ``'w'`` deletes
everything in the file (or creates it if it doesn't exist) and opens it
for writing. You can also use the mode ``'a'``. This goes to the end of
a file and adds text there. In shell terms, ``'r'`` is a bit like ``<``,
``'w'`` is a bit like ``>``, and ``'a'`` is a bit like ``>>``.

This is just the beginning of what you can do with files. If you want to
know all their methods and modes, check the official tutorial's section
on `reading and writing files`_.
File objects provide a lot of cool interfaces. These interfaces will
come back with other "file-like objects" which will come up many times
later, including in the very next section.


.. _generator function:
  https://docs.python.org/3/tutorial/classes.html#generators
.. _reading and writing files:
  https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files

CLI interfaces in Python
------------------------

Working with ``stdin``, ``stdout`` and ``stderr``
+++++++++++++++++++++++++++++++++++++++++++++++++
Unix scripting is all about filtering text streams. You have a stream
that comes from lines in a file or output of a program and you pipe it
through other programs. Unix has a bunch of special-purpose programs
just for filtering text (some of the more popular of which are
enumerated at the beginning of the previous chapter). Great cli scripts
should follow the same pattern so you can incorperate them into your
shell pipelines.  You can, of course, write your script with it's own
"interactive" interface and read lines of user input one at a time:

.. code:: Python

  username = input('What is your name? ')

This is fine in some cases, but it doesn't really promote the creation
of reusable, multi-purpose filters. With that in mind, allow me to
introduce the ``sys`` module.

The ``sys`` module has all kinds of great things as well as all kinds of
things you shouldn't really be messing with. We're going to start with
``sys.stdin``.

``sys.stdin`` is a file-like object that, you guessed it, allows you to
read from your script's ``stdin``. In Bash you'd write:

.. code:: Bash

  while read line; do # <- not actually safe. Don't use bash.
      stuff with "$line"
  done

In Python, that looks like this:

.. code:: Python

  import sys
  for line in sys.stdin:
      do_stuff_with(line) # <- we didn't remove the newline char this
                          #    time. Just mentioning it because it's a
                          #    difference between python and shell.

Naturally, you can also slurp stdin in one go -- though this isn't the
most Unix-y design choice, and you could use up your RAM with a very
large file:

.. code:: Python

  text = sys.stdin.read()

As far as stdout is concerned, you can access it directly if you like,
but you'll typically just use the ``print()`` function.


.. code:: Python

  print("Hello, stdout.")
  # ^ functionally same as:
  sys.stdout.write('Hello, stdout.\n')

Anything you print can be piped to another process. Pipelines are great.
For stderr, it's a similar story:

.. code:: Python

  print('a logging message.', file=sys.stderr)
  # or:
  sys.stderr.write('a logging message.\n')

If you want more advanced logging functions, check out the `logging
module`_.

.. _logging module:
  https://docs.python.org/3/howto/logging.html#logging-basic-tutorial

CLI Arguments
+++++++++++++
Arguments are passed to your program as a list which you can access
using ``sys.argv``. This is a bit like ``$@`` in Bash, or ``$1 $2
$3...`` etc. e.g.:

.. code:: bash

  for arg in "$@"; do
      stuff with "$arg"
  done

looks like this in Python:

.. code:: Python

  import sys
  for arg in sys.argv[1:]:
      do_stuff_with(arg)

Why ``sys.argv[1:]``? ``sys.argv[0]`` is like ``$0`` in Bash or
``argv[0]`` in C. It's the name of the executable. Just a refresher
(because you read the tutorial, right?) ``a_list[1:]`` is list-slice
syntax that returns a new list starting on the second item of
``a_list``, going through to the end.

If you want to build a more complete set of flags and arguments for a
CLI program, the standard library module for that is argparse_. The
tutorial leaves out some useful info, so here's the `API docs`_. click_
is a popular and powerful third-party module for building even more
advanced CLI interfaces.

.. _argparse: https://docs.python.org/3/howto/argparse.html
.. _API docs: https://docs.python.org/3/library/argparse.html
.. _click: http://click.pocoo.org/5/

Environment Variables and Config files
++++++++++++++++++++++++++++++++++++++
Ok, environment variables and config files aren't necessarily only part
of CLI interfaces, but they are part of the user interface in general,
so I stuck them here. Environment variables are in the ``os.environ``
mapping, so:

.. code:: Python

  >>> os.environ['HOME']
  '/home/ninjaaron'

As far as config files, in Bash, you frequently just do a bunch of
variable assignments inside of a file and source it. You can also just
write valid python files and import them as modules or eval them... but
don't do that. Arbitrary code execution in a config file is generally
not what you want.

The standard library includes configparser_, which is a parser for .ini
files, and also a json_ parser. I don't really like the idea of
human-edited json, but go ahead and shoot yourself in the foot if you
want to. At least it's flexible.

PyYaml_, the yaml parser, and toml_ are third-party libraries that are
useful for configuration files. (Install ``pyyaml`` with pip. Don't
download the tarball like the documentation suggests. I don't know why
it says that.)

.. _configparser: https://docs.python.org/3/library/configparser.html
.. _json: https://docs.python.org/3/library/json.html
.. _PyYaml: http://pyyaml.org/wiki/PyYAMLDocumentation
.. _toml: https://github.com/uiri/toml

Filesystem Stuff
----------------
Paths
+++++
So far, we've only seen paths as strings being passed to the ``open()``
function. You can certainly use strings for your paths, and the ``os``
and ``os.path`` modules contain a lot of portable functions for
manipulating paths as strings. However, since Python 3.4, we have
pathlib.Path_, a portable, abstract type for dealing with file paths,
which will be the focus of path manipulation in this tutorial.

.. code:: Python

  >>> from pathlib import Path
  >>> # make a path of the current directory
  >>> p = Path()
  >>> p
  PosixPath('.')
  >>> # iterate over directory contents
  >>> list(p.iterdir())
  [PosixPath('.git'), PosixPath('out.html'), PosixPath('README.rst')]
  >>> # use filename globbing
  >>> list(p.glob('*.rst'))
  [PosixPath('README.rst')]
  >>> # get the full path
  >>> p = p.absolute()
  >>> p
  PosixPath('/home/ninjaaron/doc/replacing-bash-scripting-with-python')
  >>> # get the basename of the file
  >>> p.name
  'replacing-bash-scripting-with-python'
  >>> # name of the parent directory
  >>> p.parent
  PosixPath('/home/ninjaaron/doc')
  >>> # split path into its parts.
  >>> p.parts
  ('/', 'home', 'ninjaaron', 'doc', 'replacing-bash-scripting-with-python')
  >>> # do some tests about what the path is or isn't.
  >>> p.is_dir()
  True
  >>> p.is_file()
  False
  # more detailed file stats.
  >>> p.stat()
  os.stat_result(st_mode=16877, st_ino=16124942, st_dev=2051, st_nlink=3, st_uid=1000, st_gid=100, st_size=4096, st_atime=1521557933, st_mtime=1521557860, st_ctime=1521557860)
  >>> # create new child paths with slash.
  >>> readme = p/'README.rst'
  >>> readme
  PosixPath('/home/ninjaaron/doc/replacing-bash-scripting-with-python/README.rst')
  >>> # open files
  >>> with readme.open() as file_handle:
  ...     pass
  >>> # make file executable with mode bits
  >>> readme.chmod(0o755)
  >>> # ^ note that octal notation is must be explicite.
  
Again, check out the documentation for more info. pathlib.Path_. Since
``pathlib`` came out, more and more builtin functions and functions in
the standard library that take a path name as a string argument can also
take a ``Path`` instance. If you find a function that doesn't, or you're
on an older version of Python, you can always get a string for a path
that is correct for your platform by by using ``str(my_path)``. If you
need a file operation that isn't provided by the ``Path`` instance,
check the docs for os.path_ and os_ and see if they can help you out. In
fact, os_ is always a good place to look if you're doing system-level
stuff with permissions and uids and so forth.

If you're doing globbing with a ``Path`` instace, be aware that, like
ZSH, ``**`` may be used to glob recursively. It also (unlike the shell)
will included hidden files (files whose names begin with a dot). Given
this and the other kinds of attribute testing you can do on ``Path``
instances, it can do a lot of of the kinds of stuff ``find`` can do.


.. code:: Python

  >>> [p for p in Path().glob('**/*') if p.is_dir()]

Oh. Almost forgot. ``p.stat()``, as you can see, returns an
os.stat_result_ instance. One thing to be aware of is that the
``st_mode``, (i.e. permissions bits) are represented as an integer, so
you might need to do something like ``oct(p.stat().st_mode)`` to show
what that number will look like in octal, which is how you set it with
``chmod`` in the shell.

.. _pathlib.Path:
  https://docs.python.org/3/library/pathlib.html#basic-use
.. _os.path: https://docs.python.org/3/library/os.path.html
.. _os: https://docs.python.org/3/library/os.html
.. _os.stat_result:
  https://docs.python.org/3/library/os.html#os.stat_result

Replacing miscellaneous file operations: ``shutil``
+++++++++++++++++++++++++++++++++++++++++++++++++++
There are certain file operations which are really easy in the shell,
but less nice than you might think if you're using python file objects
or the basic system calls in the ``os`` module. Sure, you can rename a
file with ``os.rename()``, but if you use ``mv`` in the shell, it will
check if you're moving to a different file system, and if so, copy the
data and delete the source -- and it can do that recursively without
much fuss. shutil_ is the standard library module that fills in the
gaps. The docstring gives a good summary: "Utility functions for copying
and archiving files and directory trees."

Here's the overview:

.. code:: Python
  
  >>> import shutil
  >>> # $ mv src dest
  >>> shutil.move('src', 'dest')
  >>> # $ cp src dest
  >>> shutil.copy2('src', 'dest')
  >>> # $ cp -r src dest
  >>> shutil.copytree('src', 'dest')
  >>> # $ rm a_file
  >>> os.remove('a_file') # ok, that's not shutil
  >>> # $ rm -r a_dir
  >>> shutil.rmtree('a_dir')

That's the thousand-foot view of the high-level functions you'll
normally be using. The module documentation is pretty good for examples,
but it also has a lot of details about the functions used to implement
the higher-level stuff I've shown which may or may not be interesting.
``shutil`` also has a nice wrapper function for creating zip and tar
archives with various compression algorithms, ``shutil.make_archive()``.
Worth a look, if you're into that sort of thing.

I should probably also mention ``os.link`` and ``os.symlink`` at this
point. They create hard and soft links respectively (like ``link`` and
``link -s`` in the shell). ``Path`` instances also have
``.symlink_to()`` method, if you want that.

.. _shutil: https://docs.python.org/3/library/shutil.html

Replacing ``sed``, ``grep``, ``awk``, etc: Python regex
-------------------------------------------------------
This section is not so much for experienced programmers who already know
more or less how to use regexes for matching and string manipulation in
other "normal" languages. Python is not so exceptional in this regard,
though if you're used to JavaScript, Ruby, Perl and others, you may be
surprised to find that Python doesn't have regex literals. The regex
functionally is all encapsulated in the re_ module. (The official docs
also have a `regex HOWTO`_, but it seems more geared towards people who
may not be experienced with regex.)

This section is for people who know how to use programs like ``sed``,
``grep`` and ``awk`` and wish to get similar results in Python. One
thing is that writing simple text filters in Python will never be as
elegant as it is in Perl, since Perl was more or less created to be like
a super-powered version of the ``sh`` + ``awk`` + ``sed``. The same
thing can sort of be said of ``awk``, the original text-filtering
language on Unix. The main reason to use Python for these tasks is that
the project is going to scale a lot more easily when you want to do
something a bit more complex.

One thing to be aware of is that Python is more like PCRE (Perl-style)
than BRE or ERE that most shell utilities support. If you mostly do
``sed`` or ``grep`` without the ``-E`` option, you may want to look at
the rules for Python regex (BRE is the regex dialect you know). If
you're used to writing regex for ``awk`` or ``egrep`` (ERE), Python
regex is more or less a superset of what you know. You still may want to
look at the documentation for some of the more advanced things you can
do.

.. _re: https://docs.python.org/3/library/re.html
.. _regex HOWTO: https://docs.python.org/3/howto/regex.html

How to ``grep``
+++++++++++++++
If you don't need pattern matching (i.e. something you could do with
``fgrep``), you don't need regex to match a substring. You can simply
use builtin syntax:

.. code:: python

  >>> 'substring' in 'string containing substring'
  True

Otherwise, you need the regex module to match things:

.. code:: python

  >>> import re
  >>> re.search(r'a pattern', r'string containing a pattern')
  <_sre.SRE_Match object; span=(18, 27), match='a pattern'>
  >>> re.search(r'a pattern', r'string without the pattern')
  >>> # Returns None, which isn't printed in the the Python REPL

I'm not going to go into the details of the "match object" that the
is returned at the moment. The main thing for now is that it evaluates
to ``True`` in a boolean context. You may also notice I use raw strings
``r''``. This is to keep Python's normal escape sequences from being
interpreted, since regex uses its own escapes.

So, to use these to filter through strings:

.. code:: Python

  >>> ics = an_iterable_containing_strings
  >>> # like fgrep
  >>> filtered = (s for s in ics if substring in s)
  >>> # like grep (or, more like egrep)
  >>> filtered = (s for s in ics if re.search(pattern, s))

``an_iterable_containing_strings`` here could be a list, a generator or
even a file/file-like object. Anything that will give you strings when
you iterate on it. I use `generator expression`_ syntax here instead of
a list comprehension because that means each result is produced as
needed with lazy evaluation. This will save your RAM if you're working
with a large file. You can invert the result, like ``grep -v`` simply by
adding ``not`` to the ``if`` clause. There are also flags you can add to
do things like ignore case (``flags=re.I``), etc. Check out the docs for
more.

.. _generator expression:
  https://docs.python.org/3/tutorial/classes.html#generator-expressions


How to ``sed``
++++++++++++++
Just a little tiny disclaimer: I know ``sed`` can do a lot of things and
is really a "stream editor." I'm just covering how to do substitutions
with Python, though, certainly, anything you can do with ``sed`` can
also be done in Python.

.. code:: Python

  >>> # sed 's/a string/another string/' -- i.e. doesn't regex
  >>> replaced = (s.replace('a string', 'another string') for s in ics)
  >>> # sed 's/pattern/replacement/' -- needs regex
  >>> replaced = (re.sub(r'pattern', r'replacement', s) for s in ics)

re.sub_ has a lot of additional features, including the ability to use a
*function instead of a string* for the replacement argument. I consider
this to be very useful.

.. _re.sub: https://docs.python.org/3/library/re.html#re.sub

How to ``awk``
++++++++++++++
The ``sed`` section needed a little disclaimer. The ``awk`` section
needs a bigger one. AWK is a Turing complete text-processing language.
I'm not going to cover how to do everything AWK can do with Python
idioms. I'm just going to cover the simple case of working with fields
in a line.

.. code:: Python

  >>> # awk '{print $1}'
  >>> field1 = (f[0] for f in (s.split() for s in ics))
  >>> # awk -F : '{print $1}'
  >>> field1 = (f[0] for f in (s.split(':') for s in ics))
  >>> # awk -F '[^a-zA-Z]' '{print $1}'
  >>> field1 = (f[0] for f in (re.split(r'[a-zA-Z]', s) for s in ics))

Running Processes
-----------------
in progress...

HTTP requests
-------------
also in progress...
