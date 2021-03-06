..  Copyright (c) 2014-present PlatformIO <contact@platformio.org>
    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at
       http://www.apache.org/licenses/LICENSE-2.0
    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

.. _projectconf_section_env_advanced:

Advanced options
~~~~~~~~~~~~~~~~

.. _projectconf_extra_script:

``extra_script``
^^^^^^^^^^^^^^^^

.. warning::

  This option is recommended for Advanced Users and requires Python language knowledge.

  We highly recommend to take a look at :ref:`projectconf_dynamic_build_flags`
  option where you can use any programming language. Also, this option is very
  good if you need to apply changes to the project before building/uploading
  process:

  * Macro with the latest VCS revision/tag "on-the-fly"
  * Generate dynamic headers (``*.h``)
  * Process media content before generating SPIFFS image
  * Make some changes to source code or related libraries

  More details :ref:`projectconf_dynamic_build_flags`.

.. contents::
    :local:

Allows to launch extra script using `SCons <http://www.scons.org>`_ software
construction tool. For more details please follow to "Construction Environments"
section of
`SCons documentation <http://www.scons.org/doc/production/HTML/scons-user.html#chap-environments>`_.

This option can be set by global environment variable
:envvar:`PLATFORMIO_EXTRA_SCRIPT`.

Take a look at the multiple snippets/answers for the user questions:

  - `#462 Split C/C++ build flags <https://github.com/platformio/platformio-core/issues/462#issuecomment-172667342>`_
  - `#365 Extra configuration for ESP8266 uploader <https://github.com/platformio/platformio-core/issues/365#issuecomment-163695011>`_
  - `#351 Specific reset method for ESP8266 <https://github.com/platformio/platformio-core/issues/351#issuecomment-161789165>`_
  - `#247 Specific options for avrdude <https://github.com/platformio/platformio-core/issues/247#issuecomment-118169728>`_.

Extra Linker Flags without ``-Wl,`` prefix
''''''''''''''''''''''''''''''''''''''''''

Sometimes you need to pass extra flags to GCC linker without ``Wl,``. You could
use :ref:`projectconf_build_flags` option but it will not work. PlatformIO
will not parse these flags to ``LINKFLAGS`` scope. In this case, simple
extra script will help:

``platformio.ini``:

.. code-block:: ini

    [env:env_extra_link_flags]
    platform = windows_x86
    extra_script = extra_script.py

``extra_script.py`` (place it near ``platformio.ini``):

.. code-block:: python

    Import('env')

    env.Append(
      LINKFLAGS=[
          "-static",
          "-static-libgcc",
          "-static-libstdc++"
      ]
    )

Custom Uploader
'''''''''''''''

Example, specify own upload command for :ref:`platform_atmelavr`:

``platformio.ini``:

.. code-block:: ini

    [env:env_custom_uploader]
    platform = atmelavr
    extra_script = /path/to/extra_script.py
    custom_option = hello

``extra_script.py``:

.. code-block:: python

    Import('env')
    from base64 import b64decode

    env.Replace(UPLOADHEXCMD='"$UPLOADER" ' + b64decode(ARGUMENTS.get("CUSTOM_OPTION")) + ' --uploader --flags')

    # uncomment line below to see environment variables
    # print env.Dump()
    # print ARGUMENTS

Before/Pre and After/Post actions
'''''''''''''''''''''''''''''''''

PlatformIO Build System has rich API that allows to attach different pre-/post
actions (hooks) using ``env.AddPreAction(target, callback)`` or
``env.AddPreAction(target, [callback1, callback2, ...])`` function. A first
argument ``target`` can be a name of target that is passed using
:option:`platformio run --target` command, a name of built-in targets
(buildprog, size, upload, program, buildfs, uploadfs, uploadfsota) or path
to file which PlatformIO processes (ELF, HEX, BIN, OBJ, etc.).

The example below demonstrates how to call different functions
when :option:`platformio run --target` is called with ``upload`` value.
`extra_script.py` file is located on the same level as ``platformio.ini``.

``platformio.ini``:

.. code-block:: ini

    [env:pre_and_post_hooks]
    extra_script = extra_script.py

``extra_script.py``:

.. code-block:: python

    Import("env")

    #
    # Upload actions
    #

    def before_upload(source, target, env):
        print "before_upload"
        # do some actions


    def after_upload(source, target, env):
        print "after_upload"
        # do some actions

    print "Current build targets", map(str, BUILD_TARGETS)

    env.AddPreAction("upload", before_upload)
    env.AddPostAction("upload", after_upload)

    #
    # Custom actions when building program/firmware
    #

    env.AddPreAction("buildprog", callback...)
    env.AddPostAction("buildprog", callback...)

    #
    # Custom actions for specific files/objects
    #

    env.AddPreAction("$BUILD_DIR/firmware.elf", [callback1, callback2,...])
    env.AddPostAction("$BUILD_DIR/firmware.hex", callback...)

    # custom action before building SPIFFS image. For example, compress HTML, etc.
    env.AddPreAction("$BUILD_DIR/spiffs.bin", callback...)

    # custom action for project's main.cpp
    env.AddPostAction("$BUILD_DIR/src/main.cpp.o", callback...)
