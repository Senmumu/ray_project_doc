开发小提示
================

编译
-----------

要加快编译速度，请务必通过如下方式安装Ray：

.. code-block:: shell

 cd ray/python
 pip install -e . --verbose

``-e`` 意思是可编辑，因此你对文件的更改将在不重新安装包的情况下生效。相反，如果你使用``python setup.py install``，文件将从Ray的目录复制到Python的目录（通常像是 ``/home/ubuntu/anaconda3/lib/python3.6/site-packages/ray``），这意味着你对文件的更改不会起任何作用。
如果你运行 ``pip install`` 的时候遇到 **Permission Denied** 你可以尝试添加 ``--user`` 。你可能也需要运行一些命令，像 ``sudo
chown -R $USER /home/ubuntu/anaconda3`` （代之以适当的路径）


如果你修改了C++文件，你可能需要重新编译它们。不过，你不需要重新运行 ``pip install -e .``。
替代的是，你可以用以下的办法更快地重新编译：

.. code-block:: shell

 cd ray
 bazel build //:ray_pkg

这个命令还不足以重新编译所有的C++ unit tests.如果要这么做，查看：

`本地测试`_.

调试
---------

在调试器里启动进程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

当进程崩溃时，在调试器里重新运行往往会有效。
Ray当前允许进程如下启动：

- valgrind
- the valgrind profiler
- the perftools profiler
- gdb
- tmux

To use any of these tools, please make sure that you have them installed on
your machine first (``gdb`` and ``valgrind`` on MacOS are known to have issues).
Then, you can launch a subset of ray processes by adding the environment
variable ``RAY_{PROCESS_NAME}_{DEBUGGER}=1``. For instance, if you wanted to
start the raylet in ``valgrind``, then you simply need to set the environment
variable ``RAY_RAYLET_VALGRIND=1``.

使用任一个工具，请确定你将他们在你的机器上安装好（MacOS上的``gdb`` 和 ``valgrind``众所周知是有一定问题的）
然后，你可以通过添加环境参数 ``RAY_{PROCESS_NAME}_{DEBUGGER}=1`` 启动ray的子进程，针对实例，如果你想去启动一个
raylet

To start a process inside of ``gdb``, the process must also be started inside of
``tmux``. So if you want to start the raylet in ``gdb``, you would start your
Python script with the following:

.. code-block:: bash

 RAY_RAYLET_GDB=1 RAY_RAYLET_TMUX=1 python

You can then list the ``tmux`` sessions with ``tmux ls`` and attach to the
appropriate one.

You can also get a core dump of the ``raylet`` process, which is especially
useful when filing `issues`_. The process to obtain a core dump is OS-specific,
but usually involves running ``ulimit -c unlimited`` before starting Ray to
allow core dump files to be written.

Inspecting Redis shards
~~~~~~~~~~~~~~~~~~~~~~~
To inspect Redis, you can use the global state API. The easiest way to do this
is to start or connect to a Ray cluster with ``ray.init()``, then query the API
like so:

.. code-block:: python

 ray.init()
 ray.nodes()
 # Returns current information about the nodes in the cluster, such as:
 # [{'ClientID': '2a9d2b34ad24a37ed54e4fcd32bf19f915742f5b',
 #   'IsInsertion': True,
 #   'NodeManagerAddress': '1.2.3.4',
 #   'NodeManagerPort': 43280,
 #   'ObjectManagerPort': 38062,
 #   'ObjectStoreSocketName': '/tmp/ray/session_2019-01-21_16-28-05_4216/sockets/plasma_store',
 #   'RayletSocketName': '/tmp/ray/session_2019-01-21_16-28-05_4216/sockets/raylet',
 #   'Resources': {'CPU': 8.0, 'GPU': 1.0}}]

To inspect the primary Redis shard manually, you can also query with commands
like the following.

.. code-block:: python

 r_primary = ray.worker.global_worker.redis_client
 r_primary.keys("*")

To inspect other Redis shards, you will need to create a new Redis client.
For example (assuming the relevant IP address is ``127.0.0.1`` and the
relevant port is ``1234``), you can do this as follows.

.. code-block:: python

 import redis
 r = redis.StrictRedis(host='127.0.0.1', port=1234)

You can find a list of the relevant IP addresses and ports by running

.. code-block:: python

 r_primary.lrange('RedisShards', 0, -1)

.. _backend-logging:

Backend logging
~~~~~~~~~~~~~~~
The ``raylet`` process logs detailed information about events like task
execution and object transfers between nodes. To set the logging level at
runtime, you can set the ``RAY_BACKEND_LOG_LEVEL`` environment variable before
starting Ray. For example, you can do:

.. code-block:: shell

 export RAY_BACKEND_LOG_LEVEL=debug
 ray start

This will print any ``RAY_LOG(DEBUG)`` lines in the source code to the
``raylet.err`` file, which you can find in the `Temporary Files`_.
这会打印出``RAY_LOG(DEBUG)``源码中的所有行到 ，你可以在`临时文件`_.

本地测试
---------------
假设其中一个测试（例如 ``test_basic.py`` ） 失败了。您可以在本地运行  ``python -m pytest -v python/ray/tests/test_basic.py`` 。但是，这样做会运行所有的测试，可能需要一段时间。要运行失败的特定测试，您可以执行此操作：

.. code-block:: shell

 cd ray
 python -m pytest -v python/ray/tests/test_basic.py::test_keyword_args

当运行测试时，通常第一个的测试失败是重要的，单一个测试失败通常导致同一脚本的后续测试失败。

为了编译运行C++脚本，你可以运行
.. code-block:: shell

 cd ray
 bazel test $(bazel query 'kind(cc_test, ...)')


语言分析
-------

**Running linter locally:** To run the Python linter on a specific file, run
something like ``flake8 ray/python/ray/worker.py``. You may need to first run
``pip install flake8``.
****
**在本地运行linter**: 若需要在特定文件上运行Python linter，请执行如 ``flake8 ray/python/ray/worker.py`` 类似的操作。你需要首先运行 ``pip install flake8``。
**自动格式化代码** 我们使用yapf 运行 linting， 配置文件位于 ``.style.yapf``。我们建议在格式化更改后的文件之前运行 ``scripts/yapf.sh``。注意有些项目如dataframes和rllib现在是不包含的。

.. _`issues`: https://github.com/ray-project/ray/issues
.. _`Temporary Files`: http://ray.readthedocs.io/en/latest/tempfile.html
