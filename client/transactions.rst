Transactions
-----------------

Next to the normal individual CRUD operations, the ``FhirClient`` has the ability to send transactions to a FHIR server. 
You can use transactions by calling the ``Transaction(Bundle transaction)`` function of the ``FhirClient``. This functions requires a transaction bundle as input parameter that describes the transaction the server has to carry out.
The easiest way to create a transaction bundle is by using ``TransactionBuilder``.


Using TransactionBuilder
^^^^^^^^^^^^^^^^^^^^^^^^

``TransactionBuilder`` is a function to easily create a ``Bundle`` resource to describe a transaction or a batch, instead of creating a ``Bundle`` and adding all the entries manually.
Simple interactions can be done by just calling the corresponding function of the builder e.g. (``Create()``, ``Update()``, ``Read()``, ``Delete()`` etc. ).
After you have added all your instructions all you need to do is call the ``ToBundle()`` function. This will return the transaction Bundle that you've just created, \
which you can then pass to the ``Transaction()`` function of the ``FhirClient``.

.. code:: csharp

    var pat = new Patient() { /* set up data */ };
    var client = new FhirClient("http://server.fire.ly");
    var builder = new TransactionBuilder("http://server.fire.ly", Bundle.BundleType.Transaction); // or Batch
    
    //Add a new entry to the bundle including the patient resource that needs to be created.
    builder.Create(pat);

    //Add a new entry to the bundle with instructions that Patient witd id "1337" needs to be deleted.
    builder.Delete("Patient", "1337");

    //returns the transaction bundle
    var bundle = builder.ToBundle();

    //Send the transaction to the server.
    var response = await client.TransactionAsync(bundle);

Below is the list of all interactions you can add to the transaction bundle using the ``TransactionBuilder`` :

- ``Create()``: Creates a  resource on the server.
- ``Read()``: Returns a resource from the server.
- ``Update()``: Updates a resource on the server.
- ``Delete()``:  Deletes a resources on the server.
- ``Patch()``: Patches a resource on the server.
- ``Search()``: Searches for resources on the server based on search criteria.
- ``CapabilityStatement()``: Returns the ``CapabilityStatement`` of the server.
- ``ResourceHistory()``: Returns all known historical versions of the resource from the server.
- ``CollectionHistory()``: Returns all known historical versions of resources of a specific type from the server.
- ``ServerHistory()``: Returns all known historical versions of all resources from the server.
- ``Transaction()``: Call a sub-transaction on the server. 

Conditional interactions
^^^^^^^^^^^^^^^^^^^^^^^^

Next to the 'standard' interactions, ``TransactionBuilder`` also allows you to add the conditional interactions specified by the FHIR specification. 
The conditional interactions can be added by using the optional ``SearchParam`` parameter that some of the standard interactions have. 
You can use ``SearchParam`` to specify the conditions you want.

See the example below:

.. code:: csharp

    var pat = new Patient() { /* set up data */ };
    var client = new FhirClient("http://server.fire.ly");
    var builder = new TransactionBuilder("http://server.fire.ly", Bundle.BundleType.Transaction);

    //Update the patient resource with family name "Levin", and date of birth "01-01-1990"
    var updateConditions = new SearchParams();
    updateConditions.Add("birthdate", "1990-01-01");
    updateConditions.Add("family", "Levin");
    builder.Update(updateConditions, pat); 

    //Delete the patient with email address "test@example.org"
    var deleteCondition = new SearchParams("email", "test@example.com");
    builder.Delete("Patient", deleteCondition);

     //returns the transaction bundle
    var bundle = builder.ToBundle();

    //Send the transaction to the server.
    var response = await client.TransactionAsync(bundle);

Below is the list of functions that support conditional interactions:

- ``Create()``
- ``Update()``
- ``Delete()``
- ``Patch()``