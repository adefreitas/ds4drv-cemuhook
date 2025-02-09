======
ds4drv
======

ds4drv is a Sony DualShock 4 userspace driver for Linux.

* Discussions: https://groups.google.com/forum/#!forum/ds4drv
* GitHub: https://github.com/chrippa/ds4drv
* PyPI: https://pypi.python.org/pypi/ds4drv

Features
--------

- Option to emulate the Xbox 360 controller for compatibility with Steam games
- Setting the LED color
- Reminding you about low battery by flashing the LED
- Using the trackpad as a mouse
- Custom mappings, map buttons and sticks to whatever mouse, key or joystick
  action you want
- Settings profiles that can be cycled through with a button binding

Fork information
----------------

- Added basic implementation of `cemuhook's <https://cemuhook.sshnuke.net/padudpserver.html>`_ UDP protocol.

This allows to use gyroscope, buttons and axes of DualShock 4 with `Cemu <http://cemu.info/>`_ over network or locally on any supported Linux distribution.

Implementation is quite dirty, not configurable and have only been tested with The Legend of Zelda: Breath of the Wild.

How to install
^^^^^^^^^^^^^^

The ds4drv driver is written in Python, so it can be installed using
pip.

::

   # Install pip on Debian/Ubuntu/etc.
   sudo apt-get install python3-pip

   # Install (or update to) the latest version of ds4drv-cemuhook from GitHub
   pip3 install -U https://github.com/TheDrHax/ds4drv-cemuhook/archive/master.zip

How to use
^^^^^^^^^^

The driver supports all versions of Sony DualShock 4 controllers (I use
DS4v2) connected via USB or Bluetooth.

My version of ds4drv has 5 additional command line arguments (all are
optional):

-  ``--udp`` -- starts UDP server. Without this flag ds4drv acts just
   like the official version;
-  ``--udp-host`` -- tells UDP server to what interface it should bind
   (default: 127.0.0.1);
-  ``--udp-port`` -- UDP port on which server will be listening
   (default: 26760);
-  ``--udp-no-touch`` -- do not send touchpad touches to UDP clients;
-  ``--udp-remap-buttons`` -- an option for those, who doesn’t like
   Nintendo’s button layout. It just swaps A↔B and X↔Y buttons only for
   UDP clients.

Connecting controller and starting the driver
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The driver can be started by using this command:

::

   python3 -m ds4drv --hidraw --udp

If you see a ``Permission denied`` error, you may need to copy `this`_
file to ``/etc/udev/rules.d/`` and then execute this command:
``sudo udevadm control --reload``. This udev rule allows to access the
controller from user space without root privileges. After that reconnect
your controller and try again.

You should see something like this if controller has been connected
successfully:

::

   [info][controller 1] Connected to Bluetooth Controller (AA:BB:CC:DD:FF:EE hidraw4)
   [info][controller 1] Battery: Fully charged

Configuring cemuhook
^^^^^^^^^^^^^^^^^^^^

This part is very easy. Cemuhook connects to localhost:26760 by default,
so you just need to choose the first controller (DSU1) in ``Options`` -
``GamePad motion source`` and then check the
``Also use for buttons/axes`` option in the same menu. It is also a good
idea to disable your controller in Cemu’s ``Input settings``.

|image0|

.. |image0| image:: https://i.redd.it/r9ilsyi5w1p11.png

.. _this: https://github.com/TheDrHax/ds4drv-cemuhook/blob/master/udev/50-ds4drv.rules

Installing
----------

Dependencies
^^^^^^^^^^^^

- `Python <http://python.org/>`_ 2.7 or 3.3+ (for Debian/Ubuntu you need to
  install the *python2.7-dev* or *python3.3-dev* package)
- `python-setuptools <https://pythonhosted.org/setuptools/>`_
- hcitool (usually available in the *bluez-utils* or equivalent package)

These packages will normally be installed automatically by the setup script,
but you may want to use your distro's packages if available:

- `pyudev <http://pyudev.readthedocs.org/>`_ 0.16 or higher
- `python-evdev <http://pythonhosted.org/evdev/>`_ 0.3.0 or higher


Stable release
^^^^^^^^^^^^^^

Installing the latest release is simple by using `pip <http://www.pip-installer.org/>`_:

.. code-block:: bash

    $ sudo pip install ds4drv

Development version
^^^^^^^^^^^^^^^^^^^

If you want to try out latest development code check out the source from
Github and install it with:

.. code-block:: bash

    $ git clone https://github.com/chrippa/ds4drv.git
    $ cd ds4drv
    $ sudo python setup.py install


Using
-----

ds4drv has two different modes to find DS4 devices, decide which one to use
depending on your use case.

Raw bluetooth mode
^^^^^^^^^^^^^^^^^^

Supported protocols: **Bluetooth**

Unless your system is using BlueZ 5.14 (which was released recently) or higher
it is not possible to pair with the DS4. Therefore this workaround exists,
which connects directly to the DS4 when it has been started in pairing mode
(by holding **Share + the PS button** until the LED starts blinking rapidly).

This is the default mode when running without any options:

.. code-block:: bash

   $ ds4drv


Hidraw mode
^^^^^^^^^^^

Supported protocols: **Bluetooth** and **USB**

This mode uses the Linux kernel feature *hidraw* to talk to already existing
devices on the system.

.. code-block:: bash

   $ ds4drv --hidraw


To use the DS4 via bluetooth in this mode you must pair it first. This requires
**BlueZ 5.14+** as there was a bug preventing pairing in earlier verions. How you
actually pair the DS4 with your computer depends on how your system is setup,
suggested googling: *<distro name> bluetooth pairing*

To use the DS4 via USB in this mode, simply connect your DS4 to your computer via
a micro USB cable.


Permissions
^^^^^^^^^^^

If you want to use ds4drv as a normal user, you need to make sure ds4drv has
permissions to use certain features on your system.

ds4drv uses the kernel module *uinput* to create input devices in user land and
the module *hidraw* to communicate with DualShock 4 controllers (when using
``--hidraw``), but this usually requires root permissions. You can change the
permissions by copying the `udev rules file <udev/50-ds4drv.rules>`_ to
``/etc/udev/rules.d/``.

You may have to reload your udev rules after this with:

.. code-block:: bash

    $ sudo udevadm control --reload-rules
    $ sudo udevadm trigger


Configuring
-----------

Configuration file
^^^^^^^^^^^^^^^^^^

The preferred way of configuring ds4drv is via a config file.
Take a look at `ds4drv.conf <ds4drv.conf>`_ for example usage.

ds4drv will look for the config file in the following paths:

- ``~/.config/ds4drv.conf``
- ``/etc/ds4drv.conf``

... or you can specify your own location with ``--config``.


Command line options
^^^^^^^^^^^^^^^^^^^^
You can also configure using command line options, this will set the LED
to a bright red:

.. code-block:: bash

   $ ds4drv --led ff0000

See ``ds4drv --help`` for a list of all the options.


Multiple controllers
^^^^^^^^^^^^^^^^^^^^

ds4drv does in theory support multiple controllers (I only have one
controller myself, so this is untested). You can give each controller
different options like this:

.. code-block:: bash

   $ ds4drv --led ff0000 --next-controller --led 00ff00

This will set the LED color to red on the first controller connected and
green on the second.


Known issues/limitations
------------------------

- `Bluetooth 2.0 dongles are known to have issues, 2.1+ is recommended. <https://github.com/chrippa/ds4drv/wiki/Bluetooth%20dongle%20compatibility>`_
- The controller will never be shut off, you need to do this manually by
  holding the PS button until the controller shuts off
- No rumble support


Troubleshooting
---------------

Check here for frequently encountered issues.

Failed to create input device: "/dev/uinput" cannot be opened for writing
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This could be because the uinput kernel module is not running on your
computer. Doing ``lsmod | grep uinput`` should show if the module is loaded.
If it is blank, run ``sudo modprobe uinput`` to load it. (The uinput module
needs to be installed first. Please check with your distro's package
manager.)

To have the uinput module load on startup, you can add a file
to ``/etc/modules-load.d``. For example:

.. code-block:: bash

    # in file /etc/modules-load.d/uinput.conf
    # Load uinput module at boot
    uinput

Controller not detected by game
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Some games are missing an implementation of an API that prevents DS4 from being detected, either by Steam Input, or by emulating as xboxdrv, use ~python3 -m ds4drv --hidraw --emulate-xpad instead to work around it

References
----------

The DualShock 4 report format is not open and had to be reverse engineered.
These resources have been very helpful when creating ds4drv:

- http://www.psdevwiki.com/ps4/DualShock_4
- http://eleccelerator.com/wiki/index.php?title=DualShock_4
- https://gist.github.com/johndrinkwater/7708901
- https://github.com/ehd/node-ds4
- http://forums.pcsx2.net/Thread-DS4-To-XInput-Wrapper


----

.. |dogecoin| image:: http://targetmoon.com/img/dogecoin.png
  :alt: Dogecoin
  :target: http://dogecoin.com/

|dogecoin| DCbQgDa4aEbm9QNm4ix6zYV9vMirUDQLNj
