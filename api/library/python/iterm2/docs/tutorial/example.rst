Example Script
==============

Here's the example script that iTerm2 provides for you, minus some comments:

.. code-block:: python

    #!/usr/bin/env python3

    import iterm2
    import sys

    async def main(connection, argv):
        app = await iterm2.async_get_app(connection)
        window = app.key_window
        if window is not None:
            await window.async_create_tab()
        else:
            print("No current window")

    if __name__ == "__main__":
	iterm2.connection.Connection().run(main, sys.argv)

There's a lot going on here. Let's take it part by part.

.. code-block:: python

    #!/usr/bin/env python3

This is standard boilerplate for a Python script. See :doc:`running` for
details on how scripts are run.

The next part of the template script are the imports:

.. code-block:: python

    import iterm2
    import sys

`iterm2` is a Python module (available on PyPI) that provides a nice interface
to communicate with iTerm2. The underlying implementation uses Google protobuf
and websockets. For most purposes, that is completely abstracted away.

`sys` is imported so that the script can parse command-line arguments.

.. code-block:: python

    async def main(connection, argv):

Your code goes inside `main`. The first argument is a `connection` that holds
the link to a running iTerm2 process. `main` gets called only after a
connection is established.  If the connection terminates (e.g., if you quit
iTerm2) then any attempt to use it will raise an exception and terminate your
script.

The second argument, `argv`, gives the command-line arguments passed to the
script. When you launch a script from iTerm2, it does not receive any
arguments. But you can also run this script manually and pass it command-line
arguments.

The `async` keyword may be unfamiliar if you haven't used asyncio before. It
signifies that this function can be interrupted, for example to perform a
remote procedure call over a network. Because iTerm2 communicates with the
script over a websocket connection, any time the script wishes to send or
receive information from iterm2, it will have to wait for a few milliseconds. 

The benefit of asyncio is that while the script is stopped waiting for a
response from iTerm2, other work can happen. For example, handling of
notifications from iTerm2. We'll see more about that later.

.. code-block:: python

        app = await iterm2.async_get_app(connection)

The purpose of this line is to get a reference to the :class:`iterm2.App`
object, which is useful for most things you'll want to do in a simple script.
It is a singleton that provides access to iTerm2's global state, such as its
windows.

Note the use of `await`. Any function that's defined as `async`, which most
functions in the iTerm2 API are, must be called with `await`. It means it might
not return immediately. While it waits, messages from iTerm2 or other I/O may
be received and handled.

In this case, it makes an RPC call to iTerm2 to get its state (such as the list
of windows). The returned value is an :class:`iterm2.App`.

If you forget to use `await` you'll get a warning in the Script Console.
iTerm2's library follows a naming convention to help you remember to use
`await`: any function that is declared `async` will have a name that begins
with `async_`.

.. code-block:: python

        window = app.key_window

The fetches the "key window" from the app. The key window is the window that
receives keyboard input. If iTerm2 is not active or has no windows, then no
window will be key and :meth:`iterm2.App.async_get_key_window` returns `None`.

.. code-block:: python

        if window is not None:
            await window.async_create_tab()

If there is a key window, add a tab to it. The new tab uses the default
profile.

.. code-block:: python

	else:
	    print("No current window")

This prints a diagnostic message. You can view these messages in the Script
Console. Select *Scripts > Script Console* in iTerm2 to view the output of
your scripts. If something's not working right, you can usually find the
problem in the Script Console. You can also use it to terminate a misbehaving
script.

.. code-block:: python

    if __name__ == "__main__":
	iterm2.connection.Connection().run(main, sys.argv)

The `if` statement is a bit of standard Python boilerplate; you can ignore it
as its condition will always be `True`.

The next line establishes a websocket connection to iTerm2 and then runs your
`main` function, passing it `sys.argv` which holds the command-line arguments.

Continue to the next section, :doc:`running`.

----

--------------
Other Sections
--------------

* :doc:`/index`
    * :doc:`index`
    * Example Script
    * :doc:`running`
    * :doc:`daemons`

Indices and tables
==================

* :ref:`genindex`
* :ref:`search`
