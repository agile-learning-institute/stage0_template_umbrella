# Data Standards

## Tech Stack
- MongoDB Document Database
- [MongoDB Configurator utility](https://github.com/agile-learning-institute/mongodb_configurator) to package our Schema Configurations configurations for deployment. You can use ``de up mongodb`` and visit http://localhost:8181 to see configurations.

## Standards
- All collection names are PascalCase, Singular. (TestRun not test_runs)
- All property names are snake case (first_name, street_address, etc.)
- Every collection should have the following properties
    - _id: This is a generated property but we should always be aware of it and represent it the schema. 
    - status: Status of the document - should include a "soft delete" indicator, typically an enum type.
    - name: This is a document name, should be human readable, typically of type Word
    - description: This is a text description typically of type Sentence
    - created: A tracking breadcrumb of when the document was first created.
    - saved: A tracking breadcrumb of when the document was last saved. Note the collections that are write once read many will not have this property.

## Catalog
- Review [Types](./configurator/types/) or [in the WebApp](http://localhost:8181/types) to familiarize yourself with the available types. Add new types if needed.
- Review [Enumerators](./configurator/enumerators/) or [in the WebApp](http://localhost:8181/enumerators) for a enumerators.
- Review [Collection Configurations](./configurator/configurations/) or [in the WebApp](http://localhost:8181/configurations) for a list of collections.

## Asynchronous Data Flow
Our chosen Identity Provider will provide an identity event stream, that will be persisted by the identity_api into our **Identity** collection.
