演示
===========

本演练将概述Ray的核心概念：
1.使用远程函数（tasks）[``ray.remote``]
2.获取结果（object IDs）[``ray.put``, ``ray.get``, ``ray.wait``]
3.使用远程对象 (actors) [``ray.remote``]

使用Ray，你的代码将可以在单台机器上运行，也可以轻松轻松扩展到很大的集群。为了运行这个演示，使用 ``pip install -U ray`` 安装Ray.

.. code-block:: python

  import ray

  # Start Ray. If you're connecting to an existing cluster, you would use
  # ray.init(address=<cluster-address>) instead.
  ray.init()

有关配置Ray的各种方法，请参阅`配置文档 <configure.html>`__。要启动多节点Ray群集，请参阅 `群集设置页面
<using-ray-on-a-cluster.html>`__.。你可以通过调用 ``ray.shutdown()`` 来停止Ray。要检查Ray是否已初始化，你您可以调用``ray.is_initialized()``。


远程函数 （Tasks）
------------------------


Ray支持任意Python函数异步执行。这些异步的Ray函数被称为"远程函数"。将Python函数转变成远程函数的标准方法是添加一个装饰器 ``@ray.remote`` 。这里有个示例。
 
.. code:: python

    # A regular Python function.
    def regular_function():
        return 1

    # A Ray remote function.
    @ray.remote
    def remote_function():
        return 1

这样可以造成一些表现变化：
    1. **调用：** 常规做法是使用 ``regular_function()`` 调用，而远程的版本是使用 ``remote_function.remote()`` 调用。
    2. **返回值：** ``regular_function``立即执行返回 ``1``, 而 ``remote_function`` 立即返回有个对象ID然后创建一个任务并会在一个worker进程上执行，这个结果可以使用``ray.get`` 获取。

    .. code:: python

        assert regular_function() == 1

        object_id = remote_function.remote()

        # The value of the original `regular_function`
        assert ray.get(object_id) == 1

3. **并行性:**  ``regular_function`` **接连** 调用, 例如

   .. code:: python

       # These happen serially.
       for _ in range(4):
           regular_function()

   而 ``remote_function`` 的调用 **并行**发生,例如

   .. code:: python

       # These happen in parallel.
       for _ in range(4):
           remote_function.remote()

有关如何使用``ray.remote``的详细文档，可查看 `ray.remote包参考 <package-ref.html>`__ 页。

**Object IDs** 也可以传递给远程函数。当函数实际上被执行时，**参数将被检索为常规Python对象**。例如，使用此功能：

.. code:: python

    @ray.remote
    def remote_chain_function(value):
        return value + 1


    y1_id = remote_function.remote()
    assert ray.get(y1_id) == 1

    chained_id = remote_chain_function.remote(y1_id)
    assert ray.get(chained_id) == 2


请注意以下行为：

  - 在第一个任务执行完成以前，第二个任务将不会被执行，因为第二个任务取决于第一个任务的输出。
  - 如果在不同的机器上安排两个任务，则第一个任务的输出（对应的值 ``x1_id``）将通过网络发送到第二个任务的机器。

经常地，您可能希望指定任务的资源需求（例如，一个任务可能需要GPU）。``ray.init()`` 命令可能会自动检测到可用的机器上的GPU和CPU。然而，你可以使用传递一些特定资源参数重写这个默认项，例如 ``ray.init(num_cpus=8, num_gpus=4, resources={'Custom': 2})``.

要指定一个task的CPU和GPU资源，传递 ``num_cpus`` 和 ``num_gpus`` 参数给远程装饰器。这个task会只在一台机器上运行如果那里有足够的CPU和GPU（以及其他的特定）资源可用。Ray也处理任意自定义资源。

.. 注意::

    * 如果未在 ``@ray.remote`` 装饰器里指定任何资源，则默认值为1个CPU资源，不包含其他的资源。
    * 如果指定GPU， Ray不会强制事务隔离（例如，你的task需要遵守它的请求）
    * 如果指定GPU，Ray确实以事务隔离的形式提供可见的设备（设置环境变量 ``CUDA_VISIBLE_DEVICES``），但是实际使用GPU是task的责任（例如，通过像Tensorflow或者PyTorch这样的深度学习框架进行使用。）

.. code-block:: python

  @ray.remote(num_cpus=4, num_gpus=2)
  def f():
      return 1

一个task的资源需求对Ray的调度并发性是有影响的。特别的，在一个节点上，所有正在执行的任务的总资源需求不能超过这个节点的总资源。

如下有更多的资源指定的示例
.. code-block:: python

  # Ray 也支持很小的部分的资源需求
  @ray.remote(num_gpus=0.5)
  def h():
      return 1

  # Ray 也支持自定义Custom资源
  @ray.remote(resources={'Custom': 1})
  def f():
      return 1

Further, remote function can return multiple object IDs.
此外，远程函数可以返回多个对象ID
.. code-block:: python

  @ray.remote(num_return_vals=3)
  def return_multiple():
      return 1, 2, 3

  a_id, b_id, c_id = return_multiple.remote()


Ray里的对象
--------------

在Ray里，我们可以创建并在对象上计算。我们将这些对象称为**远程对象（remote objects）**，然后我们使用**object IDs**去指向他们。远程对象被存储在**对象库（object stores）**,集群的每个节点都有个对象库。在集群设置中，我们可能无法确切地知道每个对象是在哪个机器上运行的。一个 **object ID** 本质上是可以用来指代一个远程对象的一个唯一的ID。如果你熟悉期货，我们的object IDs在概念上是相似的。


可以通过多种方式创建Object ID。

  1.它们由远程函数调用返回。
  2. 他们由``ray.put``返回。

.. code-block:: python

    y = 1
    object_id = ray.put(y)

.. autofunction:: ray.put
    :noindex:


.. 重点::

    远程对象是不可变的。也就是说，它们的值在创建后无法更改。这允许远程对象在多个对象库中进行复制，而无需同步副本。

获取结果
----------------

命令 ``ray.get(x_id)`` 获取object ID然后从对应的远程对象创建一个Python对象。对于数据这样的对象，我们可以使用共享内存来避免复制对象。

.. code-block:: python

    y = 1
    obj_id = ray.put(y)
    assert ray.get(obj_id) == 1

.. autofunction:: ray.get
    :noindex:


在启动一系列任务后，你也许想知道哪些执行完成了。这个可以由 ``ray.wait`` 完成。这个函数是如下工作的：

.. code:: python

    ready_ids, remaining_ids = ray.wait(object_ids, num_returns=1, timeout=None)

.. autofunction:: ray.wait
    :noindex:


远程类 (Actors)
-----------------------

Actors 将Ray API从函数扩展到对象。这个``ray.remote``的装饰器表示 ``Counter``类将会成为actor。一个actor本质上是一个有状态worker，每个actor在他们自己的Python进程上运行。

.. code-block:: python

  @ray.remote
  class Counter(object):
      def __init__(self):
          self.value = 0

      def increment(self):
          self.value += 1
          return self.value

为了创建几个actor，我们可以按下列方法初始化这个类

.. code-block:: python

  a1 = Counter.remote()
  a2 = Counter.remote()

实例化actor时，会发生以下事件。

1. 在集群的一个节点上启动一个Python worker进程。
2. 在该worker上实例化一个 ``Counter`` 对象.

您也可以在Actors中指定资源需求 (有关更多详细信息 请参阅`Actors部分 '
<actors.html>`__ )

.. code-block:: python

  @ray.remote(num_cpus=2, num_gpus=0.5)
  class Actor(object):
      pass

我们可以通过使用 ``.remote`` 运算符调用其方法来与actor进行交互。然后我们可以调用 ``ray.get`` 对象ID来检索实际的值。

.. code-block:: python

  obj_id = a1.increment.remote()
  ray.get(obj_id) == 1



调用在不同的actors的方法可以被并行执行，相同actor的方法调用会根据调用顺序连续执行。相同actor上的方法和其他的方法会彼此共享状态，如下所示：

.. code-block:: python

  # Create ten Counter actors.
  counters = [Counter.remote() for _ in range(10)]

  # Increment each Counter once and get the results. These tasks all happen in
  # parallel.
  results = ray.get([c.increment.remote() for c in counters])
  print(results)  # prints [1, 1, 1, 1, 1, 1, 1, 1, 1, 1]

  # Increment the first Counter five times. These tasks are executed serially
  # and share state.
  results = ray.get([counters[0].increment.remote() for _ in range(5)])
  print(results)  # prints [2, 3, 4, 5, 6]

 
要了解更多有关ray actors的信息，请参阅 `Actors section <actors.html>`__.
