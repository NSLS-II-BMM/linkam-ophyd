
Additional features of the Linkam class
=======================================

Because the ``status_code`` is such a obscure way of presenting status
information, I made a small display showing the most salient features
of the controller.

.. code-block:: python

   linkam.status()


.. _fig-statusbox:
.. figure:: _static/statusbox.png
   :target: _static/statusbox.png
   :align: center

   A quick-n-dirty status display for the Linkam

The table decodes the bits in ``linkam.status_code``.  The temperature
and set point are given.  At the top of the box, are the model numbers of the
controller and stage.

If you do this:

.. code-block:: python

   linkam.model_array.get()

the returned value is

.. code-block:: 

   array([84, 57, 54, 45, 83, 0], dtype=uint8)

an array of integers.  Huh?

The key is to recognize that the integers are ASCII code points for
the letters of the alphabet.  This translates to "T96-S", the model of
the controller.  

Here is the translation code:

.. code-block:: python
   :linenos:

      def arr2word(self, lst):
          word = ''
          for l in lst[:-1]:
              word += chr(l)
          return word
        
      @property
      def model(self):
          return self.arr2word(self.model_array.get())

Now ``model`` is a property of the class which returns the string
tranlation of the ``model_array``.


.. code-block:: python

   linkam.model

returns 'T96-S'

One of these properties exists for each of the attributes returning an
integer array.

Finally, there is a way to turn the heater stage on and off, with
versions convenient for the bsui command line and versions better
suited for use in a proper bluesky experimental plan.

.. code-block:: python
   :linenos:

      def on(self):
          self.startheat.put(1)

      def off(self):
          self.startheat.put(0)
    
      def on_plan(self):
          return(yield from mv(self.startheat, 1))

      def off_plan(self):
          return(yield from mv(self.startheat, 0))

At the bsui command line:

.. code-block:: python

   linkam.on()
   linkam.off()


In an experimental plan:

.. code-block:: python

   yield from linkam.on_plan()
   yield from linkam.off_plan()
