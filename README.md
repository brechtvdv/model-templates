# model-templates

This repository provides an explanation of the "templates" model, used for the LBLOD project.

We will explain how the model works with the IRG (interactieve reglementen generator) workflow as guideline.

# Level 0: Defining a concept

## Link with vocabulary term

A concept describes how a certain class or property (of your ontology or application profile) can be represented.

### Property

For example, to describe the location property of a traffic sign (Verkeersteken.plaatsbepaling),
we could use a concept (LocationAtStreetConcept) that describes that the sign is located at a certain street.
Each concept uses a template to format this idea into a templated text with variables.

In Turtle (RDF) we can describe this as follows:
```turtle
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>.
@prefix sh: <http://www.w3.org/ns/shacl#> .
@prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
@prefix ex: <http://example.org#> .
@prefix oslo: <http://data.vlaanderen.be/ns#> .

ex:Shape#Verkeersteken.plaatsbepaling a PropertyShape;
  sh:targetClass oslo:Verkeersteken ;
  sh:path ( oslo:plaatsbepaling ) ;
  ex:targetCanHaveConcept ex:LocationAtStreetConcept .
  
 ex:LocationAtStreetConceptA a ex:Concept, ex:LocationAtStreetConcept ;
  ex:template [
    ex:value "ter hoogte van ${straat}" ;
    ex:mapping [
      ex:variable "straat" ;
      ex:expects [
        a sh:PropertyShape ;
        sh:targetClass oslo:Adres ;
        sh:path ( oslo:straatnaam ) ;
        sh:maxCount 1 ;
        sh:datatype xsd:string 
      ]
    ]
  ] .
```

###  Class

We can create a concept for a certain class.
For example, we want to define a concept "E9a" of a traffic sign that explains that parking is allowed.
Similar to above, we use SHACL to indicate the term in the model that want to provide a concept for.

```turtle
ex:Shape#Verkeersteken a NodeShape;
  sh:targetClass oslo:Verkeersteken ;
  ex:targetCanHaveConcept ex:VerkeerstekenConcept , ex:E9aConcept .

# In general
ex:VerkeerstekenConcept a ex:Concept ;
  ex:template [
    ex:value "${wegcode}" ;
    ex:mapping [
      ex:variable "wegcode" ;
      ex:expects [
        a sh:PropertyShape ;
        sh:targetClass ex:VerkeerstekenConcept ;
        sh:path ( ex:wegcode ) ;
        sh:maxCount 1 ;
        sh:datatype xsd:string 
      ]
    ]
  ] .

# Create instantiated concepts
ex:E9aConcept a ex:VerkeerConcept ;
  ex:wegcode "Parking is allowed." .
  ex:template [
    ex:value "Parking is allowed.";
  ] .
```

This also works when E9aConcept already has been defined as a subclass of Verkeersteken:

```turtle
ex:Shape#Verkeersteken a NodeShape;
  sh:targetNode oslo:VerkeerstekenE9a ;
  ex:targetCanHaveConcept ex:E9aConcept .
```

# Link with other concepts

Our LocationAtStreetConcept can be useful with another concept, such as traffic sign E9a (E9aConcept).
A relation describes how two concepts relate with eachother.
In this case, we want to express that LocationAtStreetConcept can be used with E9aConcept.

```turtle
ex:RelationLocationAtStreetConceptAndE9a
```


