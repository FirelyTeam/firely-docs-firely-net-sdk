The CQL Packager tool
---------------------

The HL7 CQL Packager is a utility for converting CQL or ELM into other artifacts, such as C#, .NET assemblies, or FHIR Resources. It performs the following steps:

1. **CQL to ELM Conversion**: Converts CQL files into ELM JSON files (with important limitations - see disclaimer below).
2. **ELM to C#**: Converts ELM JSON files into LINQ expressions, then translates those into C# code.
3. **C# to .NET DLL Compilation**: Compiles the generated C# code into .NET assemblies.
4. **Packaging to FHIR Libraries/Measures**: Packages the assembly (and optionally the original CQL, ELM, and C# source code) into a FHIR `Library <https://hl7.org/fhir/library.html>`_ resource, creating one resource per original ELM file. Where applicable, FHIR Measure resources are also generated.

Installing the tool
^^^^^^^^^^^^^^^^^^^
The tool is distributed as a `dotnet tool`, so to install it, run:

.. code-block:: powershell

    dotnet tool install Hl7.Cql.Packager --global --prerelease

.. note::
   The ``--prerelease`` option is required because only prerelease versions are currently available.

Getting Help
^^^^^^^^^^^^
Get help for the command line tool by running any of the following commands:

.. code-block:: powershell

    cql-package --help
    cql-package cql --help
    cql-package elm --help

The two main commands
^^^^^^^^^^^^^^^^^^^^^

The CQL Packager has two main commands:

``elm`` Command
~~~~~~~~~~~~~~~

Start from ELM files and convert to one or more of the following outputs: C#, DLL, PDB, FHIR Resources.

**Usage:** ``cql-package elm [options]``

**Required Options:**

- ``--elm <directory>`` - ELM input directory containing ELM files in JSON format "\*.json"

**Output Options:**

- ``--cs <directory>`` - C# output directory for generated C# source code files "\*.g.cs"
- ``--dll <directory>`` - DLL output directory for .NET assembly libraries "\*.dll"
- ``--pdb <directory>`` - PDB output directory for portable debug symbol files "\*.pdb"
- ``--fhir <directory>`` - FHIR Resource output directory for Library and Measure files in JSON format

**FHIR-specific Options:**

- ``--cql <directory>`` - CQL input directory (REQUIRED with --fhir)
- ``--canonical-root-url <url>`` - The root canonical URL output in FHIR library
- ``--override-utc-date-time <datetime>`` - Override date output in FHIR library
- ``--json-pretty`` - Output JSON using multiline and indentation (for --fhir)

**Debug Options:**

- ``--debug-symbols <None|PortablePdb|Embedded>`` - Debug symbol generation:

  - ``None`` (DEFAULT) - No debug symbols, optimizations enabled
  - ``PortablePdb`` - Separate PDB files, no optimizations
  - ``Embedded`` - Debug symbols embedded in DLL with C# source

``cql`` Command
~~~~~~~~~~~~~~~

Start from CQL files and convert to one or more of the following outputs: ELM, C#, DLL, PDB, FHIR Resources.

**Usage:** ``cql-package cql [options]``

**Required Options:**

- ``--cql <directory>`` - CQL input directory containing CQL files "\*.cql"

**Output Options:**

- ``--elm <directory>`` - ELM output directory for generated ELM JSON files
- ``--cs <directory>`` - C# output directory for generated C# source code files "\*.g.cs"
- ``--dll <directory>`` - DLL output directory for .NET assembly libraries "\*.dll"
- ``--pdb <directory>`` - PDB output directory for portable debug symbol files "\*.pdb"
- ``--fhir <directory>`` - FHIR Resource output directory for Library and Measure files in JSON format

**FHIR-specific Options:**

- ``--canonical-root-url <url>`` - The root canonical URL output in FHIR library
- ``--override-utc-date-time <datetime>`` - Override date output in FHIR library
- ``--json-pretty`` - Output JSON using multiline and indentation (for --fhir or --elm)

**Debug Options:**

- ``--debug-symbols <None|PortablePdb|Embedded>`` - Debug symbol generation (same as elm command)

**Logging Options (both commands):**

- ``--log-append`` - Append to existing log file instead of clearing
- ``--console-log-level <level>`` - Minimum log level for console output
- ``--file-log-level <level>`` - Minimum log level for file output

Log levels: ``Critical``, ``Debug``, ``Error``, ``Information``, ``None``, ``Trace``, ``Warning``

Examples
^^^^^^^^

1. Package ELM into FHIR resources (as indented JSON) and save C# source code:

.. code-block:: powershell

    cql-package elm --elm input/elm --cql input/cql --fhir output/fhir --cs output/csharp --json-pretty

- Packages ELM JSON files from the directory ``input/elm`` into FHIR Library resources saving them to ``output/fhir``.
- The CQL files in ``input/cql`` are included in the FHIR Library resources, if they match the ELM files by file name.
- The generated C# source code is saved to directory ``output/csharp``.

2. Package CQL into FHIR resources (as indented JSON) and save C# source code:

.. code-block:: powershell

    cql-package cql --cql input/cql --fhir output/fhir --cs output/csharp --json-pretty

- Packages CQL files from the directory ``input/cql`` into FHIR Library resources saving them to ``output/fhir``.
- The ELM files are included in the FHIR Library resources, but are not saved to a separate directory.
- The generated C# source code is saved to directory ``output/csharp``.
- Read the disclaimer below.

3. Package ELM into .NET assembly DLL's, which can be stepped through in a debugger:

.. code-block:: powershell

    cql-package elm --elm input/elm --dll output/dll --debug-symbols Embedded

- Compiles ELM JSON files in the directory ``input/elm`` into .NET assemblies and saving them to ``output/dll``.
- The C# and debugging symbols are embedded in the DLLs. The DLLs are compiled without any optimizations.

4. Package ELM with portable PDB files for debugging:

.. code-block:: powershell

    cql-package elm --elm input/elm --dll output/dll --pdb output/dll --debug-symbols PortablePdb

- Compiles ELM JSON files into .NET assemblies in ``output/dll``.
- Creates separate portable PDB files in the same directory for debugging.

5. Set canonical root URL and override date for FHIR resources:

.. code-block:: powershell

    cql-package elm --elm input/elm --fhir output/fhir --canonical-root-url https://example.org/fhir/ --override-utc-date-time 2024-01-01T00:00:00Z --json-pretty

- Packages ELM into FHIR Library resources with a custom canonical root URL.
- Overrides the date timestamp in the generated FHIR resources.
- Outputs pretty-printed JSON.

Disclaimer
^^^^^^^^^^

.. warning::
   The CQL to ELM conversion process by the ``cql`` command is currently in an early stage of development and only supports basic CQL translation. **It is NOT PRODUCTION READY.**

If you encounter issues with the CQL to ELM conversion, please log them in the `Firely CQL SDK issue tracker <https://github.com/FirelyTeam/firely-cql-sdk/issues>`_.

We **HIGHLY recommend** using the ``elm`` command to package ELM files that have been generated by the CQL to ELM Java Tooling (see the `Java Tooling README <https://github.com/cqframework/clinical_quality_language/blob/master/Src/java/README.md#generate-an-elm-representation-of-cql-logic>`_).

The version of the Java Tooling that is used to generate the ELM files must be compatible with the version of the CQL Packager you are using.

Further Reading
^^^^^^^^^^^^^^^

This package is part of the `Firely CQL SDK <https://github.com/FirelyTeam/firely-cql-sdk>`_. More information can be found at `Firely's documentation site <https://docs.fire.ly/projects/Firely-NET-SDK/en/latest/cql.html>`_.
