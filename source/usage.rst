Basic Usage
===========

A great feature of :class:`ancb.NumpyCircularBuffer` is that it inherits from :class:`numpy.ndarray`.
Many features of *ndarray* are also true of *NumpyCircularBuffer*. 

Instantiation
-------------
*NumpyCircularBuffer* requires an ndarray to use for storage of data elements.

.. code-block:: python
   
   import numpy as np
   from ancb import NumpyCircularBuffer

   data = np.empty(3)
   buffer = NumpyCircularBuffer(data)

The first dimension of the ndarray used to declare a NumpyCircularBuffer is always the length 
of the buffer. For example if the buffer is (N, a, b) it would be an N length buffer of array
elements of shape (a, b).

Buffer operations
-----------------

Say we have predeclared a buffer that stores 3D vectors as rows.

.. testcode::

   import numpy as np
   from ancb import NumpyCircularBuffer
   data = np.empty((3, 3))
   buffer = NumpyCircularBuffer(data)

Appending and popping elements are done through the :func:`ancb.NumpyCircularBuffer.append` 
and the :func:`ancb.NumpyCircularBuffer.pop` functions. :func:`ancb.NumpyCircularBuffer.peek` can
be used if you don't want to consume the element at the beginnning of the buffer.

.. doctest::

   >>> buffer.append([0, 1, 2])
   >>> buffer.append([3, 4, 5])
   >>> buffer.append([6, 7, 8])
   >>> buffer
   array([[0, 1, 2],
          [3, 4, 5],
          [6, 7, 8]])

Note, while popping an element does **not** fill the space it used to fill with zeros
or anything, it simply just marks the space as available to be filled for another element.

.. doctest::

   >>> buffer.pop()
   array([0, 1, 2])
   >>> buffer.pop()
   array([3, 4, 5])
   >>> buffer.pop()
   array([6, 7, 8])
   >>> buffer
   array([[0, 1, 2],
          [3, 4, 5],
          [6, 7, 8]])

You can check if a buffer is full or empty using the useful properties
:func:`ancb.NumpyCircularBuffer.full` and :func:`ancb.NumpyCircularBuffer.empty`. 
These are convience O(1) operations that check the number of elements in the buffer 
and do a comparison to see if it's full or empty.

.. doctest::

   >>> buffer.empty
   True
   buffer.full
   False
   >>> buffer.append([0, 1, 2])
   >>> buffer.empty
   False
   >>> buffer.full
   False
   >>> buffer.append([3, 4, 5])
   >>> buffer.append([6, 7, 8])
   >>> buffer.full
   True
   >>> buffer.empty
   False

As a quick explaination of circular buffers, when you write to a full buffer, the oldest
element is overwritten.

.. doctest::

   >>> buffer.append([9, 10, 11]) 
   >>> buffer
   array([[9, 10, 11],  <- end (append will write to the next element)
          [3, 4, 5],  <- start (popping will give you this element)
          [6, 7, 8]])

Another useful property to test if you're intending on making your own wrapper functions 
is fragmentation. Roughly speaking, when the elements are no longer contigously placed 
(when the end of the buffer occurs in the data before the beginning as above), the 
buffer is said to be fragmented.

There is another O(1) operation that checks the position of the beginning and end of the buffer
along with its current size to determine if it's fragmented.

.. doctest::

   >>> buffer.fragmented
   True
   >>> buffer.append([12, 13, 14])
   >>> buffer.append([15, 16, 17])
   >>> buffer
   array([[9, 10, 11],
          [12, 13, 14],
          [15, 16, 17]])
   >>> buffer.fragmented
   False
   >>> buffer.pop()
   array([9, 10, 11])
   >>> buffer.fragmented
   False

Overloaded Operations
---------------------

While all of this is useful, perhaps what is more interesting is the idea of using
such a buffer for data processing. Let's imagine a scenario where you want to weight the
data by a vector such as [1, 0.5, 0.25] so that each element is weighted half as much as the one
before it.

If the data was coming in live, we would have no choice but to use :func:`numpy.roll` on the data
so that it aligns with our weights array. Even if we try to use a circular buffer, it turns out 
that the gains in performance by using it are lost when we are forced to roll the array 
for our algorithm since :func:`numpy.roll` has to every element in the array
and move it to a new location.

Fortunately, NumpyCircularBuffer recognizes that you shouldn't need to reorder elements
before you do the operation. Since we know where the buffer fragments, we can simply 
add the end of the buffer to the end of the array and the start of the buffer to the
start of the array at no extra cost.

All this shuffling takes place behind the scenes, so you can do:

.. doctest::

   >>> buffer.append([18, 19, 20])
   >>> buffer
   array([[18, 19, 20],
          [12, 13, 14],
          [15, 16, 17]])
   >>> buffer * np.array([0.25, 0.5, 1]).reshape(3, 1)
   array([[ 3.  ,  3.25,  3.5],
          [ 7.5 ,  8.  ,  8.5],
          [18.  , 19.  , 20.])

A Caveat: Matrix Multiplication
-------------------------------

Most of the library has no overhead; however, an exception to this are certain kinds of 
matrix multiplication. I will outline the cases below.

Right matrix multiplication (x @ buffer) if the buffer is fragmented:

- x.ndim == 1 and buffer.ndim > 1 or
- x.ndim > 1 and buffer.ndim == 1 or
- buffer.ndim == 2

Left matrix multiplication (buffer @ x) if the buffer is fragmented:

- buffer.ndim == 1

**In all of these cases, the overhead is a memory allocation of an ndarray equal to the size
of the output.** For all functions in ANCB, the specified operation takes place in two seperate
parts; however, for these kinds of matrix multiplication, the parts overlap and must be added
together for the final result unlike other functions.

The functions :func:`ancb.NumpyCircularBuffer.matmul` and :func:`ancb.NumpyCircularBuffer.rmatmul`
have been provided to combat this overhead. They allow you to use preallocated space to reduce
the overhead of the allocations for repeated operations such as in a loop.

.. testcode::

   import numpy as np
   from ancb import NumpyCircularBuffer

   data = np.empty(3)
   buffer = NumpyCircularBuffer(data)

   buffer.append(0)
   buffer.append(1)
   buffer.append(2)
   buffer.append(3)

   A = np.arange(9).reshape(3, 3)
   work_buffer = empty(3)

   # Same as A @ buffer
   print(buffer.rmatmul(A, work_buffer))

.. testoutput::

   [8 26 44]
