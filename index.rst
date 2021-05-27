..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   We rely on particular tag formats to order and present Lab images to users.  This Technote is intended to formalize and document those formats.

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

Tag formats for RSP Notebook Images
===================================

The Rubin Observatory Sciplat-Lab containers are identified via their
tags.  These tags have conventional formats understood by Cachemachine
and used to order the images and construct their display names.  For
certain types of tags, it is also possible to transform the tag data into a
`semantic-version-compatible format <https://semver.org/>`__, which
therefore makes specifying version constraints much easier.

Tag Types
---------

There are six categories of acceptible tags:

 * Alias
 * Release
 * Weekly
 * Daily
 * Release Candidate
 * Experimental
 
Anything that does not match the format of one of these tags will be
categorized as:

 * Unknown

Tag Fragments
-------------

In general, tags can be postfixed with an optional Cycle number or a
``rest`` set of one-or-more underscore-separated fields.  The Cycle is
currently unique to T&S RSP instances, and specifies the version of XML
used in this image.  It will have the form ``c<digits>.<digits>`` where
the first group of digits is the cycle number and the second group of
digits is the build iteration for that cycle, e.g. ``c0019.001``.

The ``rest`` fragment will most often encode a build datestamp, but can
be anything (but see the discussion in "Semantic Versions").

For image types for which semantic versions are supported, both of these
fields will end up packed into the ``build`` semver field.  If both are
present in a single tag, the cycle must precede ``rest``.

Recommended Tag
---------------

Finally, there is a single unique tag, known as the "recommended tag"
which is an alias tag that specifies an image that should always be
pulled and put on the front of the available-images list: it is the tag
that we think most users should be using most of the time (at a given
RSP instance).

Note on "Semantic Version"
--------------------------
Certain tag types ("Release", "Weekly", "Daily", and "Release
Candidate") can be used to construct version numbers that are
syntactically equivalent to a semantic version
``[major].[minor].[patch]`` with, perhaps, prerelease and build fields
appended.

These are **not** actually semantic versions, in that the compatibility
guarantees of true semantic versions are not present.  The version
numbers are constructed purely to aid sortability and version choosing,
because there are excellent extant tools to manipulate semantic
versions.

*Within* a given release type for a given image,
semantic versions (if supported for that type) can be used to compare
two versions of the Lab container: a higher number is more recent.
Images are
comparable *only within a tag type*.  That is, the semantic versions
from (e.g.) a weekly and a release image are not comparable.

The ``cycle`` and ``rest`` fields end up within the build field of a
semantic version.  The information in ``cycle`` is already
semver-compatible, and ``rest`` is transformed as follows: underscores
become dots, and any nonalphanumeric characters remaining are simply
dropped.
 
Alias Tags
----------

Alias tags are exactly those tags we do not expect to be stable.  There
are currently two items in this category: the recommended tag, and
``latest``, which is defined by Docker as special and is the tag-of-last
resort: if a docker image is specified without a tag, its tag is
implicitly ``latest``.

The "recommended" tag is a singleton: it is always an alias pointing
to the image that, at any moment in time, we believe most users should
be using.  Conventionally, this will be a recent weekly build, and
conventionally it has been ``recommended``.  It can be any arbitrary
string, and this will be useful, for example, to allow different
recommended images in different environments (e.g. ``recommended_idf``
versus ``recommended_summit``).

Alias tags are passed in to the tag parsing machinery from the outside
as a list of strings.  Only a tag that is an exact string match to one
of those strings will be categorized as an alias tag.

Tag Format
^^^^^^^^^^
An alias tag can be an arbitrary string (set in the cachemachine
configuration); conventionally the recommended tag has been
``recommended``.

Display Name
^^^^^^^^^^^^

The display name for an alias tag is constructed by replacing any
underscores with spaces and then titlecasing the result.  Thus
``recommended`` has the display name ``Recommended``, and if the tag
``perfectly_cromulent`` were an alias tag, it would have the display
name ``Perfectly Cromulent``.  This may be modified according to tag
resolution to include a list of other names this image is known by, for
instance, ``Recommended (Weekly 2021_20)``.

Semantic Version
^^^^^^^^^^^^^^^^
The "recommended" tag does not itself have a semantic version; however the
underlying image to which it is a pointer almost certainly does.

Notes on "recommended" category
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The "recommended" tag is significant in that it should always be
prepulled, and in that it is only an alias to another tag.  The thing
that distinguishes the tag designated as "recommended" from all others
is that it will always be prepulled, and always will be displayed at the
top of the image list in the options form.

Notes on alias tags
^^^^^^^^^^^^^^^^^^^

An alias tag requires some special handling of the tag within
cachemachine to determine what the image digest corresponding to the
recommended image is, and construct a mapping of its display name to the
actual image--this information is used both in the spawner options form
to tell the user what they're getting with and in the Lab UI to remind
the user what they're using.

Most users will use the "recommended" tag most of the time, and the
tag is generally applied to the most recent weekly build, as soon as it
has been possible to certify a weekly build by subjecting it to our QA
process.

Release
-------

Release images are the (roughly twice a year) official stack releases.
They are intended to be kept available indefinitely, and for the RSP
machinery to, at any point in time, be able to run the current release
and the two prior to it.  (If older releases are required, it may take
some work, up to and including a separately-constructed RSP instance) to
make them runnable.

Tag Format
^^^^^^^^^^

Release tags are now of the form ``r[major]_[minor]_[patch]``,
e.g. ``r21_0_1``.  Prior to Release 18, they were not
underscore-separated, e.g. ``r170``.  The first two digits are the major
version, and the last one is the minor version.  In this form, the patch
version is always 0.  Cycle and rest are permitted, so, for instance,
all of ``r_21_0_1_c0019.001``, ``r_21_0_1_20210703``, and
``r_21_0_1_c0019.001_20210703`` are allowed.

Display Name
^^^^^^^^^^^^

The display name for a release is of the form ``Release
r[major].[minor].patch``; thus ``r21_0_1`` has the display name ``Release
r21.0.1``.  Additional components (cycle and extra) are permitted and
will be appended in the following form: ``r21_0_1_c0020.002_20210703``
becomes ``Release r21.0.1 (SAL Cycle 0020, Build 002) [20210703]``.

Semantic Version
^^^^^^^^^^^^^^^^

The semantic version of a release tag is, actually,
``[major].[minor].[patch]``.  ``r21_0_1`` has version ``21.0.1``.

Cycle and build version will be added as described above.  Thus:
``r21_0_1_c0020.002_20210703`` would have the semantic version
``21.0.1+c0020.002.20210703``.

Notes on "release" category
^^^^^^^^^^^^^^^^^^^^^^^^^^^

These tags differ from other categories by not having an underscore
between the type and the release identifier.  There is no reason for
this other than historic convention.

Weekly
------

Weekly images are the bread-and-butter workhorse images.  Most users
will use the latest weekly that has been blessed as "recommended".
There are three noteworthy things about the weekly images.  First, they
are the feedstock for "recommended"; second, it is always a particular
weekly image that is chosen as the basis for a release image; and third,
we make claim that the weekly image is going to be fit-for-purpose and
therefore not utterly broken.

Tag Format
^^^^^^^^^^

Weekly tags are of the form ``w_[year]_[week]``, e.g. ``w_2021_19``.
They may have additional cycle and rest components;
``w_2021_19_c0019.001`` is an acceptable weekly tag, for instance, as is
``w_2021_19_20210513`` or indeed ``w_2021_19_c0019.001_20210513``.

Display Name
^^^^^^^^^^^^

The display name is ``Weekly [year]_[week]``; ``w_2021_19`` has the
display name ``Weekly 2021_19``.  As with releases and release
candidates, additional components are formatted and appended.  Thus
``w_2021_19_c0019.001`` would have the display name
``Weekly 2021_19 (SAL Cycle 0019, Build 001)``.

Semantic Version
^^^^^^^^^^^^^^^^

A weekly's semantic version is ``[year].[week].0``.  ``w_2021_19`` has
the version ``2021.19.0``.  Any additional components are used as the
semver ``build`` string (with underscores replaced by periods), so
``w_2021_19_c0019.001`` would become ``2021.19.0+c0019.001``.

Daily
-----

Daily images are, as the name implies, produced every night.  They are
not guaranteed to work.  They are generally used only by users needing
bleeding-edge features that haven't made it into a weekly yet.

Tag Format
^^^^^^^^^^

Daily tags are of the format ``d_[year]_[month]_[day]``; as with weekly
builds, additional underscore-separated components may exist.

Display Name
^^^^^^^^^^^^

A Daily display name is ``Daily [year]_[month]_[day]``, so
``d_2021_05_11`` becomes ``Daily 2021_05_11``.  Additional components
are handled as for weeklies.

Semantic Version
^^^^^^^^^^^^^^^^

The version for a daily image is ``[year].[month].[day]``.
``d_2021_05_11`` is simply ``2021.05.11``.  Additional components go
into the build string, as for other image types.

Release Candidate
-----------------

A release candidate follows the same rules as a release, except that it
will have one and only one additional component, ``rc[number]``, which
is an incrementing sequence number.

Tag Format
^^^^^^^^^^

The tag format is exactly that of a release format, with an additional
underscore-separated component, ``rc[number]``.  Cycle and rest are
permitted.

Display Name
^^^^^^^^^^^^

The display name resembles a Release version, except that it begins with
"Release Candidate"; the additional component will be appended with a
dash (to match the semantic version string).  ``r22_0_0_rc1`` will
have the display name ``Release Candidate r22.0.0-rc1``.

Semantic Version
^^^^^^^^^^^^^^^^

The primary components of the version are the same as release: major,
minor, patch (in general, patch will be ``0`` because it will be a
prerelease).  ``rc[number]`` will be used as the prerelease (rather than
the build) field.  Thus, ``r22_0_0_rc1`` will have the version
``22.0.0-rc1``, and ``r22_0_0_rc1_c0020.003_20210609`` would have the
version ``22.0.0-rc1+c0020.003.20210609``.

Experimental
------------

Experimental tags are used mostly by people working on the Lab machinery
itself (which is to say, mostly the author of this technote at this
point).  They start with ``exp_`` and that's really all you can say
about them (but see below).

Tag Format
^^^^^^^^^^

The experimental tag starts with ``exp_``.  In practice (and largely as
an artifact of the build process), it often looks like
``exp_[some-other-tag]_[descriptor]``, e.g. ``exp_w_2021_13_nosudo``.

Display Name
^^^^^^^^^^^^

My preference is to try the strategy hinted at above: the first word of
the display name is "Experimental", and then the rest of the tag
following ``exp_`` is fed through the display name parsing process
again; much of the time this will result in a sane display name string.
For instance ``exp_w_2021_13_nosudo`` would yield
``Experimental Weekly 2021_13 [nosudo]``.  If that re-parse fails, just
use the string following ``exp_`` as the name.  For instance
``exp_ajt_test`` would give the display name ``Experimental ajt_test``.

Semantic Version
^^^^^^^^^^^^^^^^

Experimentals will not have a semantic version string.  The only way
to sort them is lexigraphically by tag, and no temporal information is
implied.

Unknown Images
^^^^^^^^^^^^^^

Any image whose tag is not parseable according to any of the above
categories falls into an ``unknown`` type.  Fundamentally these are
handled rather like experimentals.  There is no display name separate
from the tag string, and there is no semantic version.  They have no
sort order other than lexigraphic.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
