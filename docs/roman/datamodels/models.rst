.. _datamodels:

About datamodels
================

The purpose of the data model is to abstract away the peculiarities of
the underlying file format.  The same data model may be used for data
created from scratch in memory, or loaded from ASDF files or some future file
format.

The detailed datamodel structure and specifics are contained in the
documentation included with the roman_datamodels package found here (TBD).

Each model instance is created to contain a variety of attributes and data that
are needed for analysis or to propagate information about the file and the
contents of the file. For example, the `ImageModel` class has the following
arrays associated with it:

    - `data`: The science data
    - `dq`: The data quality array
    - `err`: The error array

Along with data arrays the datamodel also contains information about the
observation that can include the observation program, exposure information,
pointing information and processing steps.

Working with models
===================

Reading a data model
--------------------

If you have an existing data file it is straightforward to access the file
using python.

.. code-block:: python

    from roman_datamodels import datamodels as rdm
    fn = 'r0019106003005004023_03203_0034_WFI01_cal.asdf'
    data_file = rdm.open(fn)
    type(data_file)

<class 'roman_datamodels.datamodels.ImageModel'>

Where the output of the type command tells you that you have imported an
ImageModel from roman_datamodels,

Creating a data model from scratch
----------------------------------

To create a new `ImageModel`, you can just::

    from roman_datamodels import datamodels as rdm
    from roman_datamodels.testing.factories import create_wfi_image
    import numpy as np

    image_node = create_wfi_image()
    image_model = rdm.ImageModel(image_node)
    type(image_model)
    #<class 'roman_datamodels.datamodels.ImageModel'>

.. warning ::
    The values in the file generated by create_wfi_image are intended to be
    clearly incorrect and should be replaced if the file is intended to be used
    for anything besides a demonstration.

Creating a data model from a file
---------------------------------

The `roman_datamodels.open` function is a convenient way to create a
model from a file on disk.  It may be passed any of the following:

    - a path to an ASDF file
    - a readable file-like object

The file will be opened, and based on the nature of the data in the
file, the correct data model class will be returned.  For example, if
the file contains 2-dimensional data, an `ImageModel` instance will be
returned.  You will generally want to instantiate a model using a
`with` statement so that the file will be closed automatically when
exiting the `with` block.

.. doctest-skip::

    from roman_datamodels import datamodels
    with datamodels.open("myimage.asdf") as im:
        assert isinstance(im, datamodels.ImageModel)

If you know the type of data stored in the file, or you want to ensure
that what is being loaded is of a particular type, use the constructor
of the desired concrete class.  For example, if you want to ensure
that the file being opened contains 2-dimensional image data::

    from roman_datamodels.datamodels import ImageModel
    with ImageModel("myimage.asdf") as im:
        # raises exception if myimage.asdf is not an image file
        pass

This will raise an exception if the file contains data of the wrong
type.

Saving a data model to a file
-----------------------------

Simply call the `save` method on the model instance.  The format to
save into will be deduced from the filename.::

    im.save("myimage.asdf")

.. note::

   This `save` always clobbers the output file.


Copying a model
---------------

To create a new model based on another model, simply use its `copy`
method.  This will perform a deep-copy: that is, no changes to the
original model will propagate to the new model::

    new_model = old_model.copy()

Looking at the contents of a model
----------------------------------

You can examine the contents of your model from within python using::

    print("\n".join("{: >20}\t{}".format(k, v) for k, v in im.items()), "\n")

which will list the contents of the ImageModel im::

    meta.aperture.name	Aperture name c1d861ddaebdb859f619fb2b79ea7bdf
    meta.aperture.position_angle	115.33996998457596
    meta.cal_step.flat_field	SKIPPED

    area	<array (unloaded) shape: [4096, 4096] dtype: float32>
    history.description	HISTORY of this file
    history.time	2021-12-29 14:03:57.465551
    history.software.name	roman_datamodels
    history.software.author	STSCI
    history.software.homepage	https://github.com/spacetelescope/roman_datamodels
    history.software.version	0.8

or you can print specifics::

    print("\n".join("{: >20}\t{}".format(k, v) for k, v in im.meta.wcsinfo.items()), "\n")
                v2_ref	1312.9491452484797
                v3_ref	-1040.7853726755036
               vparity	-1
              v3yangle	-60.0
                ra_ref	84.49289366006334
               dec_ref	-69.14101326380924
              roll_ref	0.0
              s_region	NONE

Note: These will be incorporated as methods in the data models in a future release.
