.. lsst-task-topic:: lsst.verify.tasks.commonMetrics.MemoryMetricTask

################
MemoryMetricTask
################

``MemoryMetricTask`` creates a resident set size `~lsst.verify.Measurement` based on data collected by @\ `~lsst.pipe.base.timeMethod`.
It reads the raw timing data from the top-level `~lsst.pipe.base.CmdLineTask`'s metadata, which is identified by the task configuration.

In general, it's only useful to measure this metric for the top-level task being run.
@\ `~lsst.pipe.base.timeMethod` measures the peak memory usage from process start, so the results for any subtask will be contaminated by previous subtasks run on the same data ID.

Because @\ `~lsst.pipe.base.timeMethod` gives platform-dependent results, this task may give incorrect results (e.g., units) when run in a distributed system with heterogeneous nodes.

.. _lsst.verify.tasks.MemoryMetricTask-summary:

Processing summary
==================

``MemoryMetricTask`` searches the metadata for @\ `~lsst.pipe.base.timeMethod`-generated keys corresponding to the method of interest.
If it finds matching keys, it stores the maximum memory usage as a `~lsst.verify.Measurement`.

.. _lsst.verify.tasks.MemoryMetricTask-api:

Python API summary
==================

.. lsst-task-api-summary:: lsst.verify.tasks.commonMetrics.MemoryMetricTask

.. _lsst.verify.tasks.MemoryMetricTask-butler:

Butler datasets
===============

Input datasets
--------------

``metadata``
    The metadata of the top-level command-line task (e.g., ``ProcessCcdTask``, ``ApPipeTask``) being instrumented.
    Because the metadata produced by each top-level task is a different Butler dataset type, this dataset **must** be explicitly configured when running ``MemoryMetricTask`` or a :lsst-task:`~lsst.verify.gen2tasks.MetricsControllerTask` that contains it.

Output datasets
---------------

``measurement``
    The value of the metric.
    The dataset type should not be configured directly, but should be set
    changing the ``package`` and ``metric`` template variables to the metric's
    namespace (package, by convention) and in-package name, respectively.
    Subclasses that only support one metric should set these variables
    automatically.

.. _lsst.verify.tasks.MemoryMetricTask-subtasks:

Retargetable subtasks
=====================

.. lsst-task-config-subtasks:: lsst.verify.tasks.commonMetrics.MemoryMetricTask

.. _lsst.verify.tasks.MemoryMetricTask-configs:

Configuration fields
====================

.. lsst-task-config-fields:: lsst.verify.tasks.commonMetrics.MemoryMetricTask

.. _lsst.verify.tasks.MemoryMetricTask-examples:

Examples
========

.. code-block:: py

   from lsst.verify.tasks import MemoryMetricTask

   config = MemoryMetricTask.ConfigClass()
   config.connections.metadata = "apPipe_metadata"
   config.connections.package = "pipe_tasks"
   cofig.connections.metric = "ProcessCcdMemory"
   config.target = "apPipe:ccdProcessor.runDataRef"
   task = MemoryMetricTask(config=config)

   # config.connections provided for benefit of MetricsControllerTask/Pipeline
   # but since we've defined it we might as well use it
   metadata = butler.get(config.connections.metadata)
   processCcdTime = task.run(metadata).measurement
