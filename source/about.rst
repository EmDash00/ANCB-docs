About ANCB
==========

Another NumPy Circular Buffer is another attempt to leverage Python's Numpy library
to implement the circular buffer (ring buffer) data structure. While it is relatively 
easy to implement a circular buffer (ring buffer) data structure, it is relatively 
hard to make NumPy ndarray operations work with them.

Most implementations are a class that expose append and pop methods and use NumPy
for allocating the data that backs the buffer; however, trying to use such a 
buffer with NumPy requires the same amount of overhead as other common solutions to
using a buffer such as :py:class:`collections.deque` or :py:func:`numpy.roll()`, which
reallocates new arrays when (for the case of :py:class:`collections.deque`) 
converted to a :py:class:`numpy.ndarray` or rolled into order (in the case of more common
NumPy circular buffer implementations).

This means for most other implementations that want to use NumPy ufuncs to process data,
first the data must always be copied into a newly allocated ndarray. 

This is where ANCB comes in. ANCB implements ufunc operations on a circular buffer. Unlike
other implementations, ANCB guarantees that all supported operations will never perform
extra copying or rearranging of array elements unless explicitly mentioned. An example
of addition is shown below.

[insert image here]

This reduces the overhead of allocating a new array and copying elements, which can
be significant for very large buffers or frequently used buffers in loops.

ANCB was developed primarily by Drason "Emmy" Chow during their time at IU: Bloomington
working as a Undergraduate Research Assistant. Inefficient control loops of various motion
control algorithms required continuous buffers of data that were rolled, creating large
performance bottlenecks. Work was done on ANCB to resolve such bottlenecks.
