# model-templates

This repository provides an explanation of the "templates" model, used for the LBLOD project.

We will explain how the model works with the IRG (interactieve reglementen generator) workflow as guideline.

# Overview

<img src="https://github.com/brechtvdv/model-templates/blob/main/mainv2.jpg" width="700" height="350">

The model composes four parts:

1) Shapes allow to select a property (PropertyShape:targetClass+path), a class (NodeShape:targetClass), or instance (NodeShape:targetNode).

2) Concept allow to express what is targetted with the shape. For example: the location of a traffic sign (=target of your shape) can be expressed in different ways (=concepts), such as that the traffic sign is located near a house number.

3) A concept must have at least one template, which is a textual description of what you want to express. A template can have variables and mappings for those variables. A mapping can indicate with a shape what it expects. This way, concepts that match with this shape can be suggested. 

4) Concepts have relations with other concepts. For example, a traffic sign can be related with another traffic sign, or a certain type of decision requires the use of certain articles.

# Defining a concept

## Link with vocabulary term

A concept describes how a certain class or property (of your ontology or application profile), or instance can be represented.

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
  ex:targetHasConcept ex:LocationAtStreetConcept .
  
 ex:LocationAtStreetConcept a ex:Concept ;
  ex:template [
    a ex:Template ;
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

We want to create a concept for the class "Verkeersteken".
For example, we want to define a concept "E9a" of a traffic sign that explains that parking is allowed.
Similar to above, we use SHACL to indicate the term in the model that want to provide a concept for.

```turtle
ex:Shape#Verkeersteken a sh:NodeShape;
  sh:targetClass oslo:Verkeersteken ;
  oslo:heeftVerkeersbordconcept ex:VerkeerstekenConcept ; # how it is done in OSLO
  ex:targetHasConcept ex:VerkeerstekenConcept . # how we generalize this

# General description how Verkeersteken can be described
ex:VerkeerstekenConcept a oslo:Verkeersbordconcept, ex:Concept ;
  ex:template [
    a ex:Template ;
    ex:value "${wegcode}" ;
    ex:mapping [
      ex:variable "wegcode" ;
      ex:expects [
        a sh:PropertyShape ;
        sh:targetClass oslo:Verkeersbordconcept ;
        sh:path ( ex:beschrijving ) ;
        sh:maxCount 1 ;
        sh:datatype xsd:string 
      ]
    ]
  ] .

ex:E9aVerkeersbordconcept a oslo:Verkeersbordconcept ;
  ex:beschrijving "Parking is allowed." .
  
# When your model hasn't modelled concepts
ex:Shape#Verkeersteken a sh:NodeShape;
  sh:targetClass oslo:Verkeersteken ;
  ex:targetHasConcept ex:E9aVerkeerstekenconcept .
  
ex:E9aVerkeerstekenconcept a ex:Concept ;
  ex:template [
    a ex:Template ;
    ex:value "Parking is allowed" 
  ] .
```

This also works when the concept of E9a already has been defined as a subclass of Verkeersteken by using `sh:targetNode` instead of `sh:targetClass`:

```turtle
ex:E9aVerkeersbordconcept rdfs:subClassOf oslo:Verkeersbordconcept ;

ex:Shape#Verkeersteken a NodeShape ;
  sh:targetNode oslo:E9aVerkeersbordconcept ;
  ex:targetHasConcept ex:E9aConcept .
```

## Link with other concepts

Our LocationAtStreetConcept can be useful with another concept, such as traffic sign E9a (E9aVerkeerstekenconcept).
A relation describes how two concepts relate with eachother.
In this case, we want to express that LocationAtStreetConcept can be used with E9aConcept.

```turtle
ex:E9aVerkeerstekenconcept a ex:Concept ;
  ex:relation [
    a ex:CanBeCombinedWithRelation ;
    ex:concept ex:LocationAtStreetConcept 
  ], [
    a ex:CanBeCombinedWithRelation ;
    ex:concept ex:XcConcept 
  ]
```

A second relation is added to express that bottom plate "Xc" can be used with E9a.

# Reusable templates

To create a template that works for multiple concepts, the variables in the template neeed to be dependant of the concept the template is used for.

Let's create a template for all Traffic Measures, which has two variables: a location and a traffic sign.

```turtle
ex:Shape#TrafficMeasure a sh:NodeShape;
  sh:targetClass oslo:Verkeersmaatregel ;
  ex:targetHasConcept ex:TrafficMeasureConcept .
   
ex:TrafficMeasureConcept a ex:Concept;
  ex:template ex:TrafficMeasureTemplate .
  
ex:TrafficMeasureTemplate a ex:Template ;
  ex:value "${location}\n${trafficsign}";
  ex:mapping [
    ex:variable "location" ;
    ex:expects [
      a sh:PropertyShape ;
        sh:targetClass oslo:Verkeersteken ;
        sh:path ( oslo:plaatsbepaling ) ;
        sh:maxCount 3 ; # for example
    ], [
    ex:variable "trafficsign" ;
    ex:expects [
      a sh:PropertyShape ;
        sh:targetClass oslo:Verkeersteken ;
        sh:maxCount 1 ;
    ]
  ] .
```

Here we use shapes to indicate what is expected.
This template depends on the traffic measure that needs to be installed.

When creating a specific traffic measure E9a+Xc concept:

```turtle
ex:Shape#TrafficMeasure a sh:NodeShape;
  sh:targetClass oslo:Verkeersmaatregel ;
  ex:targetHasConcept ex:E9a+XcMeasureConcept . # or maybe call it ex:targetCanHaveConcept, because multiple concepts can serve a certain target
  
ex:E9a+XcMeasureConcept a ex:Concept ;
  ex:template ex:TrafficMeasureTemplate ;
  ex:relation [
    a ex:CanBeCombinedWithRelation ;
    ex:concept ex:LocationAtStreetConcept .
  ], [
    a ex:MustUseRelation ;
    ex:concept ex:E9aVerkeerstekenconcept .
  ], [
    a ex:MustUseRelation ;
    ex:concept ex:XcVerkeerstekenconcept .
  ]
```

An application (editor, GUI) now knows throught the relations, which locationconcepts and verkeerstekenconcepts can be an answer for the template variables.

