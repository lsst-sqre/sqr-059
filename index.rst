######################################
RSP Notebook container tag conventions
######################################

.. abstract::

   The Rubin Science Platform relies on particular tag formats to order and present its Notebook Aspect images, each containing a JupyterLab UI and a particular version of the DM Pipelines processing software, to users. This technote is intended to formalize and document those formats.


Tag formats for RSP Notebook Images
===================================

The Rubin Science Platform runs JupyterLab containers which are
identified via their tags.
These tags have conventional formats understood by the Nublado controller.
The controller uses those formats to order the images, determine which should be cached (to enhance the user experience), and to construct the image name displayed to the user.


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

Primary
^^^^^^^
The primary version of a tag is derived from (and is, in most cases, identical to) the EUPS tag of the underlying DM Pipelines stack included in the image.
It is always the first component of the tag.

RSP build Counter
^^^^^^^^^^^^^^^^^

Tags may include an identifier specifying an RSP image build count. 
This field is required because the release cadence of the DM Pipelines algorithms and the JupyterLab UI and ancillary tooling may differ significantly, and we want to be able to differentiate between images with updated versions of the machinery but the same set of Pipelines software.
Further, we want to be able to index that number into release notes to document enhancements to the Notebook Aspect decoupled from Pipelines advances.

We only include this counter in Release, Release Candidate, and Experimental builds derived from one of the former two categories, although the counter continues to increment with each build regardless of type.

If specified, this goes directly after the primary component, separated by an underscore.
It takes the form ``rspX`` where ``X`` is an integer.

Note that the presence of this field, and the way it is determined, means that anyone building a local ``sciplat-lab`` image should **never** push that local build, because the build number will not be correctly incremented, even if it is set correctly for that individual build.
If a build should appear on an image repository, it is vital that it be uploaded via the GitHub action.

Cycle
^^^^^

The Cycle is currently unique to T&S RSP instances, and specifies an internal release version defining a collection of their software components.
It has the form ``c<digits>.<digits>`` where the first group of digits is the cycle number and the second group of digits is the build iteration for that cycle, e.g. ``c0019.001``.

Rest
^^^^

The ``rest`` fragment most often encodes a build datestamp, but can
be anything.
If both ``cycle`` and ``rest`` are present in a single tag, the cycle must precede ``rest``.

Recommended Tag
---------------

Finally, there is a single unique tag, known as the "recommended tag"
which is an alias tag that specifies an image that should always be
pulled and put on the front of the available-images list: it is the tag
that we think most users should be using most of the time (at a given
RSP instance).

 
Alias Tags
----------

Alias tags are exactly those tags we do not expect to be stable.
There are currently two items in this category: the recommended tag, and
``latest``, which is defined by Docker as special and is the tag-of-last
resort: if a docker image is specified without a tag, its tag is
implicitly ``latest``.
Our builds include ``latest_weekly``, ``latest_daily``, and ``latest_release`` alias tags, but the RSP does not make use of them in the spawner options form.

The "recommended" alias tag is a singleton: it is always an alias pointing
to the image that, at any moment in time, we believe most users should
be using at a given RSP instance (it may, and does, differ between instances).
Conventionally, this has historically been a recent weekly build, but in operations will generally be a recent release build.
Conventionally this tag has been the literal string ``recommended``.
It can be any arbitrary string.

Alias tags are passed in to the tag parsing machinery from the outside as a list of strings.
Only a tag that is an exact string match to one of those strings will be categorized as an alias tag.
If it is set, the "recommended" tag will always be prepended to the alias tag list.

While the ``latest`` tag is special to Docker, it is not special to the prepuller.
If the alias tag list does not include ``latest``, the ``latest`` image will only be prepulled if it is associated with an image that also has a tag indicating it should be prepulled.

Tag Format
^^^^^^^^^^
An alias tag can be an arbitrary string; conventionally the recommended tag has been the literal string ``recommended``.

Display Name
^^^^^^^^^^^^

The display name for an alias tag is constructed by replacing any
underscores with spaces and then titlecasing the result.
Thus ``recommended`` has the display name ``Recommended``, and if the tag
``perfectly_cromulent`` were an alias tag, it would have the display
name ``Perfectly Cromulent``.
This may be modified according to tag resolution to include a list of other names this image is known by, for instance, ``Recommended (Weekly 2021_20)``.

Notes on "recommended" category
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The "recommended" tag is significant in that it should always be
prepulled, and in that it is only an alias to another tag.
The thing that distinguishes the tag designated as "recommended" from all others is that it will always be prepulled, and always will be displayed at the top of the image list in the options form.
In general, most users, most of the time, should use the image designated as "recommended" at whichever RSP instance they are using.

Notes on alias tags
^^^^^^^^^^^^^^^^^^^

An alias tag requires some special handling of the tag within
the Nublado controller to determine what the image digest corresponding to the
recommended image is, and construct a mapping of its display name to the
actual image--this information is used both in the spawner options form
to tell the user what they're getting with and in the Lab UI to remind
the user what they're using.

Most users will use the "recommended" tag most of the time.

Any other defined alias tags will appear in the Nublado spawner list between "recommended" and the first of the non-alias images.

Release
-------

Release images are the official stack releases, historically on a roughly-twice-a-year-cadence.
The RSP machinery should be able to, at any point in time, run the current release and the two prior to it.
If older releases are required, it may take some work, up to and including a separately-constructed RSP instance, to make them runnable.

Historically, the "recommended" image during construction was usually a weekly; however, in operations, it will generally point to a release image.

Tag Format
^^^^^^^^^^

Release tags have the form ``r[major]_[minor]_[patch]``, e.g. ``r21_0_1``.
RSP build version, cycle and rest are permitted, so, for instance, all of ``r_21_0_1_rsp9_c0019.001``, ``r_21_0_1_c0019.001``, ``r_21_0_1_20210703``, and ``r_21_0_1_rsp9_c0019.001_20210703`` are allowed.

Display Name
^^^^^^^^^^^^

The display name for a release is of the form ``Release
r[major].[minor].patch``; thus ``r21_0_1`` has the display name ``Release
r21.0.1``.
Additional components (RSP version, cycle and extra) are permitted and will be appended in the following form: ``r21_0_1_rsp9_c0020.002_20210703``
becomes ``Release r21.0.1 (RSP Build 9) (SAL Cycle 0020, Build 002) [20210703]``.

Any image lacking an RSP version sorts lower than an image with an RSP version.
That is, "r21_0_1" will be sorted below "r21_0_1_rsp9".

Notes on "release" category
^^^^^^^^^^^^^^^^^^^^^^^^^^^

These tags differ from other categories by not having an underscore
between the type and the release identifier.  There is no reason for
this other than historic convention.

Weekly
------

While historically the "recommended" tag has been attached to a weekly image, as we pass from construction into operations, "recommended" will more usually be a release image.
The noteworthy thing about a weekly image is that it has a stronger functionality guarantee than a daily image.
We make a claim that a weekly image is going to be fit-for-purpose and therefore not utterly broken.

Tag Format
^^^^^^^^^^

Weekly tags are of the form ``w_[year]_[week]``, e.g. ``w_2021_19``.
They may have additional cycle and rest components;
``w_2021_19_c0019.001`` is an acceptable weekly tag, for instance, as is
``w_2021_19_20210513`` or indeed ``w_2021_19_c0019.001_20210513``.

Display Name
^^^^^^^^^^^^

The display name is ``Weekly [year]_[week]``; ``w_2021_19`` has the
display name ``Weekly 2021_19``.
As with releases and release candidates, additional components are formatted and appended.
Thus ``w_2021_19_c0019.001`` would have the display name
``Weekly 2021_19 (SAL Cycle 0019, Build 001)``.


Daily
-----

Daily images are, as the name implies, produced every night.
They are not guaranteed to work.
They are generally used only by users needing bleeding-edge features that haven't made it into a weekly yet.

Tag Format
^^^^^^^^^^

Daily tags are of the format ``d_[year]_[month]_[day]``; as with weekly
builds, additional underscore-separated components may exist.

Display Name
^^^^^^^^^^^^

A Daily display name is ``Daily [year]_[month]_[day]``, so
``d_2021_05_11`` becomes ``Daily 2021_05_11``.
Additional components are handled as for weeklies.


Release Candidate
-----------------

A release candidate follows the same rules as a release, except that it
will have one and only one additional component, ``rc[number]``, which
is an incrementing sequence number.

Tag Format
^^^^^^^^^^

The tag format is exactly that of a release format, with an additional
underscore-separated component, ``rc[number]``.
RSP build version, cycle and rest are permitted.

Display Name
^^^^^^^^^^^^

The display name resembles a Release version, except that it begins with
"Release Candidate"; the additional component will be appended with a
dash.
``r22_0_0_rc1`` will have the display name ``Release Candidate r22.0.0-rc1``.
As with Release versions, these will be sorted within a version by RSP build number, and a build lacking such a number will be sorted below all builds with numbers.


Experimental
------------

Experimental tags are used mostly by people working on the Lab machinery
itself (which is to say, mostly the author of this technote at this
point).
They start with ``exp_`` but usually have enough resemblance to other tag types that meaningful display names can be constructed.

Tag Format
^^^^^^^^^^

The experimental tag starts with ``exp_``.
In practice (and largely as an artifact of the build process), it usally looks like ``exp_[some-other-tag]_[descriptor]``, e.g. ``exp_w_2021_13_nosudo``.
This is the preferred format (and the one produced by the GitHub Action build for an experimental image), although any tag that starts with ``exp_`` is a legal experimental tag.

Display Name
^^^^^^^^^^^^

The first word of the display name is "Experimental", and then the rest of the tag following ``exp_`` is fed through the display name parsing process again; much of the time this will result in a sane display name string.
Our GitHub Actions-based build process produces experimental tags of this format and therefore display names will generally be legible.
For instance, ``exp_w_2021_13_nosudo`` yields ``Experimental Weekly 2021_13 [nosudo]``.
If that re-parse fails, we just use the string following ``exp_`` as the name.
For instance, ``exp_ajt_test`` gives the display name ``Experimental ajt_test``.
Sorting of experimental images is purely lexigraphic, because we cannot guarantee that the tag will be parseable as an experimental derived from some more structured version.

Unknown Images
^^^^^^^^^^^^^^

Any image whose tag is not parseable according to any of the above categories falls into an ``unknown`` type.
Fundamentally these are handled rather like experimentals.

Implementation
--------------

Within the Nublado controller, these conventions are implemented in the `RSPImageTag class <https://github.com/lsst-sqre/nublado/blob/main/controller/src/controller/models/domain/rsptag.py>`_.

RSP build number
^^^^^^^^^^^^^^^^

The implementation we have settled on is the `$GITHUB_RUN_NUMBER <https://docs.github.com/en/actions/reference/workflows-and-actions/variables>`_ variable available within GitHub Actions.

We settled on this after considering several choices.

One alternative was to use a semantic version from the `sciplat-lab <https://github.com/lsst-sqre/sciplat-lab>`_ repository.
However, this doesn't capture what we want, because sciplat-lab dependencies are floating, and therefore behavioral changes may occur without changes to the repository.
Further, we have not been following semver for sciplat-lab releases and don't have any particular desire to.
For those reasons, semantic versioning was inappropriate.

We could also have retagged sciplat-lab with an ascending number (as a Git tag) each build, or perhaps each build within the Release or Release candidate categories.
This would have had the advantage of having build versions be contiguous and keeping the numbers smaller.
However, it would have required development of a custom GitHub Action to perform this task.
We additionally are not confident, at least not without extensive testing, that having thousands of git tags applied to commits within a repository would not create performance problems when retrieving and parsing the tags.

As of August 2025, the build number is in the 2000s.
We expect to build one image a day over the lifetime of the project; that should be on the order of 4000 more images from our standard build cadence.
Since we build experimental images relatively infrequently, and will probably build them with decreasing frequency as the project matures, it is unlikely the build number will reach 10,000 during the lifetime of the project.
A more realistic concern is that we change build systems or repository names in such a way that the build number is reset; in that case we can simply add a constant offset to keep the numbers monotonic.

Deriving "semantic versions" from RSP tags
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Certain tag types ("Release", "Weekly", "Daily", and "Release
Candidate") can be used to construct version numbers that are
syntactically equivalent to a semantic version
``[major].[minor].[patch]`` with, perhaps, prerelease and build fields
appended.

These are **not** actually semantic versions, in that the compatibility
guarantees of true semantic versions are not present.
The version numbers are constructed purely to aid sortability and version choosing, because there are excellent extant tools to manipulate semantic versions.

*Within* a given release type for a given image, semantic versions (if supported for that type) can be used to compare two versions of the Lab container: a higher number is more recent.
Images are comparable *only within a tag type*.
That is, the semantic versions from (e.g.) a weekly and a release image are not comparable.

The ``cycle`` and ``rest`` fields end up within the build field of a
semantic version.
The information in ``cycle`` is already in a lexigraphical format compatible with semver's ``build`` field, and ``rest`` is transformed as follows: underscores become dots, and any nonalphanumeric characters remaining are simply dropped.

Sort priority as implemented by the Nublado controller therefore does **not** follow semver, as it does take the build field into account when sorting, which is not permitted under semver.
Additionally, the Nublado controller considers RSP build number when constructing a sort order.
The RSP build number does not contribute to the derived semver string in any way.

Alias
"""""

Alias tags do not have semantic versions, although the underlying image tag they point to almost certainly does.

Release
"""""""

The semantic version of a release tag is, ``[major].[minor].[patch]``.  ``r21_0_1`` has version ``21.0.1``.

Cycle and build version will be added as described above.
Thus: ``r21_0_1_c0020.002_20210703`` would have the semantic version ``21.0.1+c0020.002.20210703``.

Weekly
""""""

A weekly's semantic version is ``[year].[week].0``.  ``w_2021_19`` has
the version ``2021.19.0``.  Any additional components are used as the
semver ``build`` string (with underscores replaced by periods), so
``w_2021_19_c0019.001`` would become ``2021.19.0+c0019.001``.

Daily
"""""

The version for a daily image is ``[year].[month].[day]``.
``d_2021_05_11`` is simply ``2021.05.11``.
Additional components go into the build string, as for other image types, and similarly, the RSP version (if any) is not reflected in the image's semantic version.

Release candidate
"""""""""""""""""

The primary components of the version are the same as release: major,
minor, patch (in general, patch will be ``0`` because it will be a
prerelease).  ``rc[number]`` will be used as the prerelease (rather than
the build) field.
Thus, ``r22_0_0_rc1`` will have the version ``22.0.0-rc1``, and ``r22_0_0_rc1_c0020.003_20210609`` would have the version ``22.0.0-rc1+c0020.003.20210609``.
As with releases, although RSP build information will be present and used to determine sort order, it will not contribute towards the semantic version.

Experimental
""""""""""""

If parsing of an experimental tag reveals structure that can be read as a tag that admits a derived semantic version, that semantic version will be attached to the experimental image.
However, the semantic version is not used for sorting purposes in the dropdown menu on the JupyterHub spawner.
This is for two reasons: first, we cannot guarantee the tag will be parseable in that manner, and second, since semantic versions are not comparable across tag types, comparing the versions of, e.g., a release-based experimental and a weekly-based experimental would be meaningless.

Unknown
"""""""

Unknown images do not have a semantic version string.  The only way to
sort them is lexigraphically by tag, and no temporal information is
implied.

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
