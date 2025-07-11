

.. _corpus_structure:

****************************
Corpus formats and structure
****************************

Prior to running the aligner, make sure the following are set up:

1. A pronunciation dictionary for your language should specify the pronunciations of orthographic transcriptions.

2. The sound files to align.

3. Orthographic annotations in .lab files for individual sound files (:ref:`prosodylab_format`)
   or in TextGrid intervals for longer sound files (:ref:`textgrid_format`).

.. note::

   A collection of preprocessing scripts to get various corpora of other formats is available in the :xref:`mfa_reorg_scripts` and :xref:`corpus_creation_scripts`.

Specifying speakers
===================

The sound files and the orthographic annotations should be contained in one directory structured as follows::

    +-- textgrid_corpus_directory
    |   --- recording1.wav
    |   --- recording1.TextGrid
    |   --- recording2.wav
    |   --- recording2.TextGrid
    |   --- ...

    +-- prosodylab_corpus_directory
    |   +-- speaker1
    |       --- recording1.wav
    |       --- recording1.lab
    |       --- recording2.wav
    |       --- recording2.lab
    |   +-- speaker2
    |       --- recording3.wav
    |       --- recording3.lab
    |   --- ...


.. _speaker_characters_flag:

Using :code:`--speaker_characters` flag
---------------------------------------

.. warning::

   In general I would not recommend using this flag and instead sticking to the default behavior of per-speaker directories for :ref:`prosodylab_format` and per-speaker tiers for :ref:`textgrid_format`.

MFA also has limited support for a flat structure directory structure where speaker information is encoded in the file like::

    +-- prosodylab_corpus_directory
    |   --- speaker1_recording1.wav
    |   --- speaker1_recording1.lab
    |   --- speaker1_recording2.wav
    |   --- speaker1_recording2.lab
    |   --- speaker2_recording3.wav
    |   --- speaker2_recording3.lab
    |   --- ...

By specifying :code:`--speaker_characters 8`, then the above files will be assigned "speaker1" and "speaker2" as their speakers from the first 8 characters of their file name.  Note that because this is dependent on the number of initial characters, if your speaker codes have varying lengths, then it will not behave correctly.

For historical reasons, MFA also supports reading speaker information from files like the following::

    +-- prosodylab_corpus_directory
    |   --- experiment1_speaker1_recording1.wav
    |   --- experiment1_speaker1_recording1.lab
    |   --- experiment1_speaker1_recording2.wav
    |   --- experiment1_speaker1_recording2.lab
    |   --- experiment1_speaker2_recording3.wav
    |   --- experiment1_speaker2_recording3.lab
    |   --- ...

By specifying :code:`--speaker_characters prosodylab`, then the above files will be assigned "speaker1" and "speaker2" from the second element when splitting by underscores in the file name.

.. _single_speaker_flag:

Using :code:`--single_speaker` flag
-----------------------------------

MFA uses multiple jobs to process utterances in parallel.  The default setup assigns utterances to jobs based on speakers, so all utterances from ``speaker1`` would be assigned to Job 1, all utterances from ``speaker2`` would be assigned to Job 2, and so on.  However, if there is only one speaker in the corpus (say if you're generating alignments for a Text-to-Speech corpus), then all files would be assigned to Job 1 and only one process would be used.  By using the :code:`--single_speaker` flag, MFA will distribute utterances across jobs equally and it will skip any speaker adaptation steps.


Transcription file formats
==========================

In addition to the sections below about file format, see :ref:`text_normalization` for details on how the transcription text is normalized for dictionary look up, and :ref:`configuration_dictionary` for how this normalization can be customized.

.. _prosodylab_format:

Prosodylab-aligner format
-------------------------

Every audio file you are aligning must have a corresponding .lab
file containing the text transcription of that audio file.  The audio and
transcription files must have the same name. For example, if you have ``givrep_1027_2_1.wav``,
its transcription should be in ``givrep_1027_2_1.lab`` (which is just a
text file with the .lab extension).

.. note::

   If you have transcriptions in a tab-separated text file (or an Excel file, which can be saved as one), you can generate .lab files from it using the relabel function of relabel_clean.py. The `relabel_clean.py script <https://github.com/prosodylab/prosodylab.alignertools/blob/master/relabel_clean.py>`_ is currently in the `prosodylab.alignertools repository on GitHub <https://github.com/prosodylab/prosodylab.alignertools>`_.

If no ``.lab`` file is found, then the aligner will look for any matching ``.txt`` files and use those.

In terms of directory structure, the default configuration assumes that
files are separated into subdirectories based on their speaker (with one
speaker per file).

An alternative way to specify which speaker says which
segment is to :ref:`speaker_characters_flag` with some number of characters of the file name as the speaker identifier.

The output from aligning this format of data will be TextGrids that have a tier
for words and a tier for phones.

.. _textgrid_format:

TextGrid format
---------------

The other main format that is supported is long sound files accompanied
by TextGrids that specify orthographic transcriptions for short intervals
of speech.


    .. figure:: ../_static/librispeech_textgrid.png
        :align: center
        :alt: Input TextGrid in Praat with intervals for each utterance and a single tier for a speaker

If :ref:`speaker_characters_flag`, the tier names will not be used as speaker names, and instead the first X characters
specified by the flag will be used as the speaker name.

By default, each tier corresponds to a speaker (speaker "237" in the above example), so it is possible to
align speech for multiple speakers per sound file using this format.


    .. figure:: ../_static/multiple_speakers_textgrid.png
        :align: center
        :alt: Input TextGrid in Praat with intervals for each utterance and tiers for each speaker

Stereo files are supported as well, where it assumes that if there are
multiple talkers, the first half of speaker tiers are associated with the first
channel, and the second half of speaker tiers are associated with the second channel.

The output from aligning will be a TextGrid with word and phone tiers for
each speaker.

    .. figure:: ../_static/multiple_speakers_output_textgrid.png
        :align: center
        :alt: TextGrid in Praat following alignment with interval tiers for each speaker's words and phones

.. note::

   Intervals in the TextGrid less than 100 milliseconds will not be aligned.

.. _reference_alignment_format:

Including reference alignments
==============================

As of version 3.3, it's possible to use reference alignments that have been hand corrected or verified to guide training
or adaptation of acoustic models.  The two ways to specify reference alignments, either in the corpus files or supplied via
a ``--reference_directory`` argument supplied to :ref:`train_acoustic_model` or :ref:`adapt_acoustic_model`. In both cases,
MFA will use a tier named like ``{speaker_name} - phones`` (i.e., the same format as MFA's TextGrid output). MFA will also
load intervals from ``{speaker_name} - words`` as reference words, but these are not actually used or necessary, as all alignments
are on a phone basis.

.. note::

    You can use MFA's :xref:`anchor` to more easily fix and save manual alignments for use in MFA training.

Sound files
===========

The default format for sound files in Kaldi is ``.wav``.  However, if MFA is installed via conda, you should have :code:`sox` and/or :code:`ffmpeg` available which will pipe sound files of various formats to Kaldi in wav format.  Running :code:`sox` by itself will a list of formats that it supports. Of interest to speech researchers, the version on conda-forge supports non-standard :code:`wav` formats, :code:`aiff`, :code:`flac`, :code:`ogg`, and :code:`vorbis`.

.. note::

   ``.mp3`` files on Windows are converted to wav via ``ffmpeg`` rather than ``sox``.

   Likewise, :code:`opus` files can be processed using ``ffmpeg`` on all platforms

   Note that formats other than ``.wav`` have extra processing to convert them to ``.wav`` format before processing, particularly on Windows where ``ffmpeg`` is relied upon over ``sox``.  See :ref:`wav_conversion` for more details.

Sampling rate
-------------

Feature generation for MFA uses a consistent frequency range (20-7800 Hz).  Files that are higher or lower sampling rate than 16 kHz will be up- or down-sampled by default to 16 kHz during the feature generation procedure, which may produce artifacts for upsampled files.  You can modify this default sample rate as part of configuring features (see :ref:`feature_config` for more details).

Bit depth
---------

Kaldi can only process 16-bit WAV files.  Higher bit depths (24 and 32 bit) are getting more common for recording, so MFA will automatically convert higher bit depths via :code:`sox` or :code:`ffmpeg`.

Duration
--------

In general, audio segments (sound files for Prosodylab-aligner format or intervals for the TextGrid format) should be less than 30 seconds for best performance (the shorter the faster).  We recommend using breaks like breaths or silent pauses (i.e., not associated with a stop closure) to separate the audio segments.  For longer segments, setting the beam and retry beam higher than their defaults will allow them to be aligned.  The default beam/retry beam is very conservative 10/40, so something like 400/1000 will allow for much longer sequences to be aligned.  Though also note that the higher the beam value, the slower alignment will be as well.  See :ref:`configuration_global` for more details.
