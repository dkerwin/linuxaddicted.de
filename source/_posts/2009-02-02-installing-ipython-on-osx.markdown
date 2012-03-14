--- 
layout: post
title: Installing iPython on OSX
tags: 
- Howto
- MacOSX
- Python
- Scripts
status: publish
type: post
published: true
meta: 
  _edit_lock: "1233579332"
  _edit_last: "2"
---
If you don't know iPython by now: [check it out](http://ipython.scipy.org/moin/)

It's a pretty nice tool if you work with python from the terminal. Especially the easy way to get information about modules and functions. The easiest way to install (especially in a Mac environment) is to use the "alldeps" tarball.

<!--more-->

{% codeblock lang:bash %}
deathstar-mac ~ $ wget http://ipython.scipy.org/dist/alldeps/ipython-alldeps-0.9.1.tar
deathstar-mac ~ $ tar xvf ipython-alldeps-0.9.1.tar
deathstar-mac ~ $ cd ipython-alldeps-0.9.1.tar
deathstar-mac ~ $ make
{% endcodeblock %}

The default install location is ~/usr/local. You may change this by modifying the install scripts but for now use the default. Now it's time to make some modifications to your profile. Add the following lines:

{% codeblock lang:bash %}
deathstar-mac ~ $ export PATH="$PATH:/Users/YOUR_USERNAME/usr/local/bin"
deathstar-mac ~ $ export PYTHONPATH=$PYTHONPATH:/Users/YOUR_USERNAME/usr/local/lib/python2.5/site-packages
{% endcodeblock %}

Let's complete the installation process:

{% codeblock lang:bash %}
deathstar-mac ~ $ iptest
deathstar-mac ~ $ cd ipython-alldeps-0.9.1/ipython-0.9.1
deathstar-mac ~ $ python setup.py install --prefix=~/usr/local

deathstar-mac ~ $ ipython
Leopard libedit detected.
Python 2.5.1 (r251:54863, Apr 15 2008, 22:57:26) 
Type "copyright", "credits" or "license" for more information.

IPython 0.9.1 -- An enhanced Interactive Python.
?         -> Introduction and overview of IPython's features.
%quickref -> Quick reference.
help      -> Python's own help system.
object?   -> Details about 'object'. ?object also works, ?? prints more.

In [1]: 
{% endcodeblock %}

Enjoy!

