Basic Usage
===========

A great feature of :class:`NumpyCircularBuffer` is that it inherits from :class:`numpy.ndarray`.
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

Appending and popping elements are done through the :func:`NumpyCircularBuffer.append` 
and the :func:`NumpyCircularBuffer.pop` functions. :func:`NumpyCircularBuffer.peek` can
be used if you don't want to consume the element at the beginnning of the buffer.

.. doctest::

   >>> buffer.append([0, 1, 2])
   >>> buffer.append([3, 4, 5])
   >>> buffer.append([6, 7, 8])
   >>> buffer
   >>> array([[0, 1, 2],
              [3, 4, 5],
              [6, 7, 8]])

You can check if a buffer is full or empty using useful properties. These are convience O(1)
operations that check the number of elements in the buffer and do a comparison to see if it's
full or empty.

.. doctest::

   >>> buffer.full
   >>> True
   >>> buffer.empty
   >>> False

As a quick explaination of circular buffers, when you write to a full buffer, the oldest
element is overwritten

.. doctest::

   >>> buffer.append([9, 10, 11]) 
   >>> buffer
   >>> array([[9, 10, 11],
              [3, 4, 5],
              [6, 7, 8]])

Another useful property to test if you're intending on making your own wrappers is fragmentation.
Rough speaking, when the elements are no longer continuously placed (such as when the end 
of the buffer occurs in the data before the beginning), the buffer is said to be fragmented.

This is another O(1) operation that checks the position of the beginning and end of the buffer
along with its current size to determine if its fragmented.

.. doctest::

   >>> buffer.fragmented
   >>> True

Overloaded Operations
---------------------

While all of this is useful, perhaps what is more interesting is the idea of using
such a buffer for data processing. Let's imagine a scenario where you want to weight the
data by a vector such as [0.25, 0.5, 0.1] so that older data is weighted less, each older
one weighted half as much as the one before it.

If the data was coming in live, we would have no choice but to use :func:`numpy.roll` on the data
so that it aligns with our weights array. So it turns out that we really didn't gain a lot
by trying to use a circular buffer since :func:`numpy.roll` has to every element in the array
and move it to a new location.

Fortunately, NumpyCircularBuffer recognizes that you shouldn't need to reorder elements
before you do the operation. Since we know where the buffer fragments, we can simply 
add the end of the buffer to the end of the array and the start of the buffer to the
start of the array at no extra cost.

All this shuffling takes place behind the scenes, so you can do:

.. doctest::

   >>> buffer * np.array([1, 0.5, 0.1]).reshape(3, 1)
   >>> array([[0.75, 1., 1.25],
              [3, 3.5, 4.],
              [9., 10, 11.])

