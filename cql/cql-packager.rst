The CQL Packager tool
---------------------

The CQL Packager tool turns ELM files into C#. There are 

- Converting ELM to C#
- Creating FHIR `Library <https://hl7.org/fhir/library.html>`_ resources containing the CQL, ELM, C# and compiled code.

The tool cannot, as of now, convert CQL into ELM, for this you need to use the existing `Java-based tooling <https://github.com/cqframework/clinical_quality_language/blob/master/Src/java/README.md#generate-an-elm-representation-of-cql-logic>`_.

Installing the tool
^^^^^^^^^^^^^^^^^^^
The tool is distributed as a `dotnet tool`, so to install it, run 

.. code-block:: powershell

    dotnet tool install Hl7.Cql.Packager --global

You can add ``--prerelease`` and ``--version`` to install a specific (prerelease) version of the tool.

Running the tool
^^^^^^^^^^^^^^^^
Run the tool without arguments to display a help text:

.. code-block:: powershell

    cql-package

There are the main commandline arguments to pass into the tool:

- ``--elm``: an existing directory where the ELM files can be found. These files must have the ``.json`` suffix.
- ``--cql``: an existing directory where the corresponding original CQL files can be found. These must have the ``.cql`` suffix.
- ``--fhir``: a directory where the produced FHIR Library resources will be written to (in FHIR Json format). Though optional, without this argument you will not get any useful output. If the directory does not exist, it will be created.
- ``--cs``: a directory where the generated C# will be written to. If the directory does not exist, it will be created.

For example, if you have checked out the Demo project in the repo, running the tool like so will produce both libraries and source code:

.. code-block:: powershell

    cql-package --elm Elm\json --cql Cql\input --fhir c:\packager-output-fhir --cs c:\packager-output-cs

Note that if the tool encounters any unparseable files, it will neatly report a stacktrace and exit.
