FAQ
===

Below you'll find frequently asked questions, performance tips, and other
topics that don't really fit in to the rest of the SatPy documentation.

If you have any other questions that aren't answered here feel free to make
an issue on GitHub or talk to us on the Slack team or mailing list. See the
:ref:`contributing <dev_help>` documentation for more information.

.. contents:: Topics
    :depth: 1
    :local:

Why is SatPy slow on my powerful machine?
-----------------------------------------

SatPy depends heavily on the dask library for its performance. However,
on some systems dask's default settings can actually hurt performance.
By default dask will create a "worker" for each logical core on your
system. In most systems you have twice as many logical cores
(also known as threaded cores) as physical cores. Managing and communicating
with all of these workers can slow down dask, especially when they aren't all
being used by most SatPy calculations. One option is to limit the number of
workers by doing the following at the **top** of your python code:

.. code-block:: python

    import dask
    from multiprocessing.pool import ThreadPool
    dask.config.set(pool=ThreadPool(8))
    # all other SatPy imports and code

This will limit dask to using 8 workers. Typically numbers between 4 and 8
are good starting points. Number of workers can also be set from an
environment variable before running the python script, so code modification
isn't necessary:

.. code-block:: bash

    DASK_NUM_WORKERS=4 python myscript.py

Similarly, if you have many workers processing large chunks of data you may
be using much more memory than you expect. If you limit the number of workers
*and* the size of the data chunks being processed by each worker you can
reduce the overall memory usage. Default chunk size can be configured in SatPy
by setting the following environment variable:

.. code-block:: bash

    export PYTROLL_CHUNK_SIZE=2048

This could also be set inside python using ``os.environ``, but must be set
**before** SatPy is imported. This value defaults to 4096, meaning each
chunk of data will be 4096 rows by 4096 columns. In the future setting this
value will change to be easier to set in python.

Why multiple CPUs are used even with one worker?
------------------------------------------------

Many of the underlying Python libraries use math libraries like BLAS and
LAPACK written in C or FORTRAN, and they are often compiled to be
multithreaded. If necessary, it is possible to force the number of threads
they use by setting an environment variable:

.. code-block:: bash

    OMP_NUM_THREADS=2 python myscript.py

What is the difference between number of workers and number of threads?
-----------------------------------------------------------------------

The above questions handle two different stages of parallellization: Dask
workers and math library threading.

The number of Dask workers affect how many separate tasks are started,
effectively telling how many chunks of the data are processed at the same
time. The more workers are in use, the higher also the memory usage will be.

The number of threads determine how much parallel computations are run for
the chunk handled by each worker. This has minimal effect on memory usage.

The optimal setup is often a mix of these two settings, for example

.. code-block:: bash

    DASK_NUM_WORKERS=2 OMP_NUM_THREADS=4 python myscript.py

would create two workers, and each of them would process their chunk of data
using 4 threads when calling the underlying math libraries.

How do I avoid memory errors?
-----------------------------

If your environment is using many dask workers, it may be using more memory
than it needs to be using. See the "Why is SatPy slow on my powerful machine?"
question above for more information on changing SatPy's memory usage.
