About ANCB
==========

Another Numpy Circular Buffer is another attempt to leverage Python's Numpy library
to implement the circular buffer (ring buffer) data structure. While it is relatively 
easy to implement a circular buffer (ring buffer) data structure, it is relatively 
hard to make Numpy ndarray operations work with them.

Most implementations are a class that expose append and pop methods and use Numpy
for allocating of the data that backs the buffer; however, trying to use such a 
buffer with Numpy requires the same amount of overhead as other common solutions to
using a buffer such as :py:class:`collections.deque` or :py:func:`numpy.roll()`, which
in short reallocate new arrays when (for the case of :py:class:`collections.deque`) 
converted to a :py:class:`numpy.ndarray` or rolled into order (in the case of 
:py:class:`numpy.ndarray`).

This means for most other implementations that want to use Numpy ufuncs to process data,
first the data must always be copied into a newly allocated array. 

This is where ANCB comes in. ANCB implements operations on a circular buffer. Unlike
other implementations, ANCB guaruntees that all supported operations will never perform
extra copying or rearranging of array elements unless explicitly mentioned. An example
of addition is shown below.

[insert image here]

This reduces the overhead of allocating a new array and copying elements, which can
be significant for very large buffers.

ANCB was developed primarily by Drason "Emmy" Chow during their time at IU: Bloomington
working as a Undergraduate Research Assistant. Inefficient control loops of various motion
control algorithms required continuous buffers of data that were rolled, creating large
performance bottlenecks. Work was done on ANCB to resolve such bottlenecks.
