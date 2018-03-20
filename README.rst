Replacing Bash Scripting with Python
====================================

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

  $ for item in "${my_array[@]}"; do stuff with "$item"; done

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
programming, we start this tutorial the basics of working with text
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
          ## the .rstrip() method is optional. It removes trailing
          ## whitespace from the line.

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

and then you use it like:

.. code:: Python

  for line in iter_with(open('my_file.txt')):
      do_stuff_with(line)

``yield from`` means it's a `generator function`_, and it's
handing over control to a sub-iterator (the file object, in this case)
until that iterator runs out of things to return. Don't worry if that
doesn't make sense. It's a more advanced Python topic and not necessary
for administrative scripting.

.. _generator function: https://docs.python.org/3/tutorial/classes.html#generators

If you don't want to iterate on lines, which is the most memory
efficient way to deal with text files, you can slurp entire contents of
a file at once like this:

.. code:: Python

  with open('my_file.txt') as my_file:
      file_text = my_file.read()
      ## or
      lines = list(my_file)
      ## or with newline characters removed
      lines = my_file.read().splitlines()

      ## This code wouldn't actually run because the file hasn't been
      ## rewound to the beginning after it's been read through.


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
know all their methods and modes, check the official tutorial_.
File objects provide a lot of cool interfaces. These interfaces will
come back with other "file-like objects" which will come up many times
later, including in the very next section.

.. _tutorial: https://docs.python.org/3/tutorial/inputoutput.html#reading-and-writing-files

CLI interfaces in Python
------------------------

Working with ``stdin``, ``stdout`` and ``stderr``
+++++++++++++++++++++++++++++++++++++++++++++++++
Unix scripting is all about filtering text streams. You have a stream
that comes from lines in a file or output of a program, and you pipe it
through other programs. It has a bunch of special-purpose programs just
for filtering text (some of the more popular of which are enumerated at
the beginning of the previous chapter). Great cli scripts should follow
the same pattern so you can incorperate them into your shell pipelines.
You can, of course, write your script with it's own "interactive"
interface and read lines of user input one at a time:

.. code:: Python

  username = input('What is your name? ')

This is fine in some cases, but it doesn't really promote the creation
of reusable, multi-purpose filters. With that in mind, allow me to
introduce the ``sys`` module.

The ``sys`` module has all kinds of great things as well as all kinds of
things you shouldn't really be messing with. We're going to start with
``sys.stdin``.

``sys.stdin`` is a file-like object that, you guessed it, allows you to
read from your scripts ``stdin``. In Bash you'd write:

.. code:: Bash

  while read line; do # <- not actually safe
      stuff with "$line"
  done

In Python, that looks like this:

.. code:: Python

  import sys
  for line in sys.stdin:
      do_stuff_with(line) # <- we didn't remove the newline char this
                          #    time. Just mentioning it because it's a
                          #    difference between python and shell

Naturally, you can also slurp stdin in one go -- though this isn't the
most Unix-y design choice, and you could use up your RAM with a very
large file:

.. code:: Python

  text = sys.stdin.read()

As far as stdout is concerned, you can access it directly if you like,
but you'll typically just use the ``print()`` function.


.. code:: Python

  sys.stdout.write('Hello, stdout.\n')
  # ^ functionally same as:
  print("Hello, stdout.")

..
  Anything you print can be piped to another process. Pipelines are great.
  For stderr, it's a similar story:


