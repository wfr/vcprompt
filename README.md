vcprompt: version control information in your prompt
====================================================

vcprompt is a little C program that prints a short string with
barebones information about the current working directory for various
version control systems. You configure your shell to include the
output of vcprompt in your prompt, and you get version control
information in your prompt.

vcprompt is designed to be small and lightweight rather than
comprehensive. Currently, it has varying degrees of support for
Mercurial, Git, Subversion, CVS, and Fossil working copies.

vcprompt has minimal dependencies: it does as much as it can with the
standard C library and POSIX calls. It should work on any
POSIX-compliant system with a C99 compiler. Some optional features
require external libraries (see "Dependencies" below).

To build vcprompt from the source tarball:

    ./configure
    make

If you're building in a source checkout, you also need GNU autoconf:

    autoconf
    ./configure
    make

(vcprompt requires GNU make, so if you are using a BSD variant where
the default make is BSD make, you will need to install GNU make and
run "gmake".)

To install it in your home directory:

    make install PREFIX=$HOME

To make life easier for packagers, the Makefile also supports DESTDIR:

    make install DESTDIR=/tmp/packageroot PREFIX=/usr

Please report build failures to the development mailing list,
vcprompt-devel@googlegroups.com.

vcprompt includes a fairly comprehensive test suite. If you want to
run it, see "Testing" below.


Dependencies
============

vcprompt requires GNU make to build.

vcprompt requires GNU autoconf to build from a source checkout (but
not from a source tarball).

Support for Subversion >= 1.7 requires SQLite 3. If it's not present on
the build system, vcprompt will support Subversion <= 1.6. Either way,
the build should succeed and the tests should pass. To install the
required files:

    sudo apt-get install libsqlite3-dev   # Debian, Ubuntu
    sudo yum install libsqlite3x-devel    # Fedora, Red Hat
    
If you have multiple versions of SQLite installed (this can be a
problem on Mac OS X), you might need to specify the installation
prefix of the one you want -- e.g., to use the brew package:

    ./configure --with-sqlite3=/usr/local
    
To see which features are built-in to your vcprompt binary, run

    ./vcprompt -F
    

Configuration
=============

(For more details, see the man page.)

To use it with bash, just call it in PS1:

    PS1='\u@\h $(vcprompt)\$ '
    
To use it with zsh, you need to enable shell option PROMPT_SUBST, and
then do similarly to bash:

    setopt prompt_subst
    PROMPT='[%n@%m] [%~] $(vcprompt)'
    

Format Strings
==============

You can customize the output of vcprompt using format strings, which
can be specified either on the command line or in the VCPROMPT_FORMAT
environment variable. For example:

    vcprompt -f "%b"
    
and

    VCPROMPT_FORMAT="%b" vcprompt
    
are equivalent.

Format strings use printf-like "%" escape sequences:

    %n  name of the VC system managing the current directory
        (e.g. "cvs", "hg", "git", "svn")
    %b  current branch name
    %r  current revision
    %u  ? if there are any unknown files
    %m  + if there are any uncommitted changes (added, modified, or
        removed files)
    %%  a single % character
    
All other characters are expanded as-is.

(For more details, see the man page.)


Testing
=======

To run vcprompt's test suite:

    make check
    
Test failures should be loud and obvious. Please report any test
failures to the development mailing list:

    vcprompt-devel@googlegroups.com.
    
To check for memory errors, you can run vcprompt's test suite under
valgrind:

    make grind
    
Obviously, this requires that you have valgrind installed.

Testing multiple versions of the same tool
------------------------------------------

Subversion changes its working copy format every couple of years, so
vcprompt supports three formats: the pre-1.4 XML format, the 1.4..1.6
plain-text format, and the post-1.7 SQLite format. Actually testing
these requires that you have different versions of Subversion on hand,
each installed in a separate prefix.

For example, I keep multiple versions in /usr/local/subversion-1.x, so
I can test them like this:

    rm -f tests/svn-repo*.tar && make check-svn TOOLPATH=/usr/local/subversion-1.6/bin
    rm -f tests/svn-repo*.tar && make check-svn TOOLPATH=/usr/local/subversion-1.7/bin
    
Actually *building* multiple versions of Subversion is harder than you
would believe. (In fact, I've been unable to build anything older than
1.5, so vcprompt's support for pre-1.4 working copies is currently
untested.)

TOOLPATH is supported for all tools; I also keep multiple versions of
Mercurial around, so I can test vcprompt against them:

    rm -f tests/hg-repo.tar && make check-svn TOOLPATH=/usr/local/mercurial-2.4/bin
    rm -f tests/hg-repo.tar && make check-svn TOOLPATH=/usr/local/mercurial-2.5/bin
    [...etc...]
    

Contributing
============

Patches are welcome.  Please follow these guidelines:

  * Ensure that the tests pass before and after your patch. To run the
    tests quickly:

        make check
        
    To run the tests using valgrind (detect memory leaks):

        make grind
        
    If you cannot run the tests on a POSIX-compliant system, that is a
    bug: please let me know.

  * If at all possible, add a test whenever you fix a bug or implement
    a feature. If you can write a test that has no dependencies (e.g.
    no need to execute "git" or "hg" or whatever), add it to
    tests/test-simple. Otherwise, add it to the appropriate
    VC-specific test script, e.g. tests/test-git if it needs to be
    able to run git.

  * Keep the dependencies minimal: preferably just C99 and POSIX. If
    you need to run an external executable, make sure it makes sense:
    e.g. it's OK to run "git" in a git working directory, but *only*
    if we already know we are in a git working directory.

  * Performance matters! I wrote vcprompt so that people wouldn't have
    to spawn and initialize an entire Python or Perl interpreter every
    time they get a new shell prompt. Using system() to turn around
    and spawn external commands -- especially ones that involve a
    relatively large runtime penalty like Python scripts -- misses the
    point of vcprompt.

    In fact, you'll find that vcprompt contains hacky little
    reimplementations of select bits and pieces of Mercurial, git,
    Subversion, and CVS precisely in order to avoid running external
    commands. (And, in the case of Subversion, to avoid depending on a
    large, complex library.)

  * Stick with my coding style:
    - 4 space indents
    - no tabs
    - curly braces on the same line *except* when defining a function
    - C99 comments and variable declarations are OK, at least until
      someone complains that their compiler cannot handle them

  * Feel free to add yourself to the contributors list below. (If you
    don't do it, I'll probably forget.)


Author Contact
==============

vcprompt was written by Greg Ward <greg at gerg dot ca>.

The latest version is available from either of my public Mercurial
repositories:

[http://hg.gerg.ca/vcprompt/](http://hg.gerg.ca/vcprompt/)

[http://bitbucket.org/gward/vcprompt/](http://hg.gerg.ca/vcprompt/)


Other Contributors
==================

In chronological order:

    Daniel Serpell
    Jannis Leidel
    Yuya Nishihara
    KOIE Hidetaka
    Armin Ronacher
    Jordi Fita
    Gregg Lind
    Jakob Kramer
    Robson Roberto Souza Peixoto
    Alexandre Carmel-Veilleux
    Felix Eckhofer

Thanks to all!


Copyright & License
===================

Copyright (C) 2009-2014, Gregory P. Ward and contributors.

This program is free software; you can redistribute it and/or modify
it under the terms of the GNU General Public License as published by
the Free Software Foundation; either version 2 of the License, or
(at your option) any later version.

This program is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
GNU General Public License for more details.

You should have received a copy of the GNU General Public License along
with this program; if not, write to the Free Software Foundation, Inc.,
51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
