
Using ``IReadOnlyDictionary`` with POCOs
----------------------------------------
All POCOs in the SDK implement the ``IReadOnlyDictionary`` interface, through which the POCO's data can be accessed dynamically. 
This interface is implemented explicitly, so in order to use it, you should cast a POCO to this interface:

.. code-block:: csharp

	Patient p = new Patient() {   };
	IReadOnlyDictionary<string,object> patientDict = p;

As is clear from the generic parameters, the dictionary is keyed by a string: the element's name. The dictionary will "contain" the key
if the underlying element is not null, and (in the case of collection elements) has at least one item.

The value found for the key in the dictionary is of type ``object``, but can really only contain a restricted set of datatypes:

* A .NET primitive type (string, int, long), a byte array or a DateTimeOffset
* Another ``IReadOnlyDictionary<string,object>``
* A collection of the types from the previous buttons

Resources and datatypes (like ``HumanName``, ``Identifier``, but also ``FhirString`` and the backbone types like ``ContactComponent`` in ``Patient``)
are represented using ``IReadOnlyDictionary``. You can navigate down the tree by using the indexer or the ``TryGetValue()`` function of the interface. 
Continuing the example above:

.. code-block:: csharp

	IReadOnlyDictionary<string,object> activeDict = patientDict["active"];
	// since the underlying type is of course a FhirBoolean you could also do:
	FhirBoolean active = (FhirBoolean)activeDict;

Note that, in contrast to the Json serialization of FHIR, resources will not contain additional magic `resourceType` entry in their dictionary.

Datatypes come in two flavours in FHIR: complex datatypes (and their sub-type, the anonymous backbone types like
``Patient.ContactComponent``) and primitives. In FHIR, even primitives are complex since they not only
contain a primitive value, but also possibly an ``extension`` and/or an ``id``. All datatypes are represented as a normal nested ``IReadOnlyDictionary``, but
the primitives have two additional features. First off, the dictionary for a FHIR primitive has a key ``value`` that contain the actual .NET primitive
for the value of a FHIR primitive (see the table below for details). Additionally, these FHIR primitives all implement ``IFhirPrimitive``, an interface
that contains a property ``ObjectValue`` of type ``object``. This property can be used to retrieve the .NET-based primitive value of the FHIR primitive as well:

.. code-block:: csharp

   object val1 = activeDict["value"];
   object val2 = ((IFhirPrimitive)activeDict).ObjectValue;
   Assert.AreEqual(val1,val2);

This is even true for narrative XML (found in each resource's ``Text`` property), which is represented using the ``XHtml`` datatype in the dictionary.
even though ``Text.Div`` is of type string in the POCO itself:

.. code-block:: csharp

   Narrative narrative = p.Text;
   string xmlDiv1 = narrative.Div;
   IReadOnlyDictionary<string,object> narrativeDict = narrative;
   
   IReadOnlyDictionary<string,object> divDict = narrative["div"];
   Assert.IsTrue(divDict is XHtml);
   Assert.IsTrue(divDict is IFhirPrimitive);

   object xmlDiv2 = divDict["value"];
   object xmlDiv3 = ((IFhirPrimitive)divDict).ObjectValue;
   Assert.AreEqual(xmlDiv1,xmlDiv2);
   Assert.AreEqual(xmlDiv2,xmlDiv3);

This is in line with the definition of ``Narrative`` in `the specification <http://hl7.org/fhir/narrative.html#Narrative>`_,
but the way we generated the ``Narrative`` datatype as a POCO slightly diverges from the specification, so this is worth pointing out.

The table below lists the exact .NET type used for the representation of primitives:

.. list-table::
 :header-rows: 1

 * - POCO Type
   - .NET type
 * - Base64Binary
   - byte[]
 * - Code
   - string
 * - Date/Time/FhirDateTime
   - string
 * - FhirBoolean
   - bool?
 * - FhirDecimal
   - decimal?
 * - FhirString 
   - string
 * - FhirUri/FhirUrl
   - string
 * - Id/Oid/Uuid/Canonical
   - string
 * - Instant
   - DateTimeOffset?
 * - Integer/PositiveInt/UnsignedInt
   - int?
 * - Integer64
   - long?
 * - Markdown
   - string
 * - XHtml (Narrative.Div)
   - XHtml
 * - Code<T>
   - string
 
Choice elements are present in the dictionary both by their "normal" unsuffixed name (i.e. ``onset`` for ``Condition.onset``, not ``onsetPeriod`` for example).

.. code-block:: csharp

   Condition c = new Condition { OnSet = new FhirDateTime() };
   IReadOnlyDictionary<string,object> conditionDict = c;

   var onset1 = conditionDict["onset"];
   Assert.IsTrue( onset1 is FhirDateTime );
    
   Assert.IsTrue( conditionDict.ContainsKey("onset") );
   Assert.IsFalse( conditionDict.ContainsKey("onsetDateTime") );
   Assert.IsFalse( conditionDict.ContainsKey("onsetString") );

Since the classes for the resources and datatypes implement ``IReadOnlyDictionary<string,object>`` they also implement ``IEnumerable<KeyValuePair<string,object>>``.
Similarly, this enumeration will contain the unsuffixed name for such choice elements.
