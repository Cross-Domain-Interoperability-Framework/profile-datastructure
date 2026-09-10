# CDIF Data Structure Profile — Implementation Guide

The **CDIF Data Structure profile** (`cdifDataStructure`) documents the *organization of values inside a dataset* — how variables play roles in a data structure (dimension, measure, identifier, attribute), how the variables are keyed (primary and foreign keys), and which physical data layout is in use (wide, long, dimensional). It builds on the CDIF Data Description profile (which describes the variables themselves) and adds the structural relationships among them. The intention is to represent a data structure as a reusable object that can be referenced from multiple dataset descriptions.

This profile is combined with core, discovery and data description in this document specification:

- **doc-discoverydatadescriptionstructure** — core + discovery + data description + data structure.

This guide documents the profile's added constraints; consult that repository for end-to-end examples.

See also [graphical presentation of the Data Structure profile model](https://cross-domain-interoperability-framework.github.io/metadataBuildingBlocks/cdif-uml-model/CDIFDataStructure/index.html).

# Table of contents

- [1. Purpose and scope](#1-purpose-and-scope)
- [2. Conformance](#2-conformance)
- [Properties added to schema:DataDownload](#properties-added-to-schemadatadownload)
  - [cdi:isStructuredBy](#cdiisstructuredby)
- [Classes added by this profile](#classes-added-by-this-profile)
  - [cdi:DataStructure (abstract)](#cdidatastructure-abstract)
  - [cdi:DimensionalDataStructure](#cdidimensionaldatastructure)
  - [cdi:LongDataStructure](#cdilongdatastructure)
  - [cdi:WideDataStructure](#cdiwidedatastructure)
  - [cdif:DataStructureComponent (abstract)](#cdifdatastructurecomponent-abstract)
  - [cdif:AttributeComponent](#cdifattributecomponent)
  - [cdif:DimensionComponent](#cdifdimensioncomponent)
  - [cdif:IdentifierComponent](#cdifidentifiercomponent)
  - [cdif:MeasureComponent](#cdifmeasurecomponent)
  - [cdif:VariableDescriptorComponent](#cdifvariabledescriptorcomponent)
  - [cdif:VariableValueComponent](#cdifvariablevaluecomponent)
  - [cdi:DimensionGroup](#cdidimensiongroup)
  - [cdif:PrimaryKey](#cdifprimarykey)
  - [cdif:ForeignKey](#cdifforeignkey)
  - [cdif:RepresentedVariable](#cdifrepresentedvariable)
  - [schema:Identifier](#schemaidentifier)
- [Classes referenced from other CDIF profiles](#classes-referenced-from-other-cdif-profiles)
  - [dcat:CatalogRecord](#dcatcatalogrecord)
  - [schema:DataDownload](#schemadatadownload)
  - [schema:DefinedTerm](#schemadefinedterm)
  - [cdif:UnitType](#cdifunittype)
  - [cdifConceptOrTerm](#cdifconceptorterm)
  - [cdif:DescriptorVariable](#cdifdescriptorvariable)
  - [cdif:DescriptorValueDomain](#cdifdescriptorvaluedomain)
  - [cdi:SubstantiveValueDomain](#cdisubstantivevaluedomain)
  - [cdi:SentinelValueDomain](#cdisentinelvaluedomain)
  - [cdif:EnumerationDomain](#cdifenumerationdomain)
  - [CdifCodelistConcept](#cdifcodelistconcept)
  - [cdif:ValueAndConceptDescription](#cdifvalueandconceptdescription)
- [Validation](#validation)
- [Provenance of the artifacts](#provenance-of-the-artifacts)

## 1. Purpose and scope

[↑ Back to TOC](#table-of-contents)

A CDIF Data Description record (the parent profile) describes *what variables* a dataset measures and *how they map to physical positions in the file*. The Data Structure profile adds another layer: *what roles those variables play in the structure of the dataset and how records are keyed*. A schema:DataDownload distribution gains a `cdi:isStructuredBy` link that points to one of three concrete structure types — `cdi:WideDataStructure`, `cdi:LongDataStructure`, or `cdi:DimensionalDataStructure` — each of which lists data-structure components (identifier, measure, attribute, dimension, variable-descriptor, variable-value). Components are typed by their role and bind to a `cdif:RepresentedVariable` (a logical variable, independent of physical layout). Primary and foreign keys reference ordered sets of represented variables.


## 2. Conformance

[↑ Back to TOC](#table-of-contents)

An instance document that includes this profile must declare conformance:

```json
"schema:subjectOf": {
  "@type": ["schema:CreativeWork"], "schema:additionalType": ["dcat:CatalogRecord"],
  "dcterms:conformsTo": [
    "https://w3id.org/cdif/data_structure/1.1"
  ]
}
```

and must include at least one `schema:distribution` of type `schema:DataDownload` whose `cdi:isStructuredBy` references a concrete `cdi:DataStructure`.

Other properties in this profile are optional; conformance requires only that the constraints expressed in `cdifDataStructureStructuredSchema.json` and `dataStructureRules.shacl` are satisfied. The schema is open-world: properties not described by the profile are allowed.

# Properties added to schema:DataDownload

[↑ Back to TOC](#table-of-contents)

The Data Structure profile adds one property to each `schema:DataDownload` distribution item.

## cdi:isStructuredBy

- **Cardinality:** Required on conforming DataDownload items.
- **Content:** one of: an inline `cdi:DimensionalDataStructure`, `cdi:LongDataStructure`, or `cdi:WideDataStructure`; or an `@id` reference to a DataStructure object defined elsewhere (but accessible on the web).
- **Description:** Links a distribution to the data structure that organizes its records.

# Classes added by this profile

## cdi:DataStructure (abstract)

[↑ Back to TOC](#table-of-contents)

- Abstract base for `cdi:DimensionalDataStructure`, `cdi:LongDataStructure`, and `cdi:WideDataStructure`. Instances are always typed as one of those subtypes; this class is never instantiated directly. It carries the common properties listed below; the subtypes constrain which `DataStructureComponent` kinds may appear in `cdi:has_DataStructureComponent`.

### @type

- **Cardinality:** Required
- **Content:** array of string. Concrete value is one of `cdi:DimensionalDataStructure`, `cdi:LongDataStructure`, `cdi:WideDataStructure`.

### @id

- **Cardinality:** Optional
- **Content:** string
- **Description:** Identifier for this DataStructure node, useful when the structure is referenced from multiple distributions.

### cdi:has_DataStructureComponent

- **Cardinality:** Required (≥1)
- **Content:** array of `cdif:DataStructureComponent` subtype instances or `@id` references.
- **Description:** The components that make up the data structure. The valid component subtypes depend on the concrete DataStructure subtype — see each subtype below.

### cdi:has_PrimaryKey

- **Cardinality:** Optional
- **Content:** inline `cdif:PrimaryKey` or `@id` reference.
- **Description:** Variables in the structure that uniquely identify a record.

### cdi:has_ForeignKey

- **Cardinality:** Optional
- **Content:** array of inline `cdif:ForeignKey` or `@id` references.
- **Description:** Variables whose values identify records in a different dataset.

## cdi:DimensionalDataStructure

[↑ Back to TOC](#table-of-contents)

- Structure of a multidimensional ("cube") dataset, described by dimension, measure and attribute components. Subtype of `cdi:DataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdi:DimensionalDataStructure`.

### cdi:has_DataStructureComponent

- **Cardinality:** Required (≥1)
- **Content:** array; each item is one of `cdif:DimensionComponent`, `cdif:MeasureComponent`, `cdif:AttributeComponent`.

### cdi:has_DimensionGroup

- **Cardinality:** Optional
- **Content:** array of `cdi:DimensionGroup`.
- **Description:** Groups of dimensions that together address a coordinate position in the cube.

(Inherits `cdi:has_PrimaryKey` and `cdi:has_ForeignKey` from `cdi:DataStructure`.)

## cdi:LongDataStructure

[↑ Back to TOC](#table-of-contents)

- Structure of a long ("entity-attribute-value") dataset, described by identifier, measure, attribute, variable-descriptor and variable-value components. Subtype of `cdi:DataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdi:LongDataStructure`.

### cdi:has_DataStructureComponent

- **Cardinality:** Required (≥1)
- **Content:** array; each item is one of `cdif:IdentifierComponent`, `cdif:VariableDescriptorComponent`, `cdif:VariableValueComponent`, `cdif:AttributeComponent`.

( cdi:LongDataStructure inherits `cdi:has_PrimaryKey` and `cdi:has_ForeignKey` from `cdi:DataStructure`.)

## cdi:WideDataStructure

[↑ Back to TOC](#table-of-contents)

- Structure of a wide (one-row-per-unit) dataset, described by identifier, measure and attribute components. Each record represents properties of one unit in the population. Subtype of `cdi:DataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdi:WideDataStructure`.

### cdi:has_DataStructureComponent

- **Cardinality:** Required (≥1)
- **Content:** array; each item is one of `cdif:IdentifierComponent`, `cdif:MeasureComponent`, `cdif:AttributeComponent`.

(Inherits `cdi:has_PrimaryKey` and `cdi:has_ForeignKey` from `cdi:DataStructure`.)

## cdif:DataStructureComponent (abstract)

[↑ Back to TOC](#table-of-contents)

- Abstract base for the role-typed components that make up a `cdi:DataStructure`. Instances are always typed as one of the concrete subtypes below. Every component binds a `cdif:RepresentedVariable` to a role inside the structure.

## cdif:AttributeComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable in the context of a data structure to qualify observations or provide other supplementary information. Permitted in all three concrete DataStructure subtypes.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:AttributeComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isDefinedBy_RepresentedVariable

- **Cardinality:** Optional
- **Content:** `cdif:RepresentedVariable` inline or `@id` reference. Logical variable that contains values for this component.

### cdi:qualifies

- **Cardinality:** Optional
- **Content:** array of inline `cdif:DataStructureComponent` or `@id` references.
- **Description:** Other components that this attribute qualifies.

### cdi:identifier

- **Cardinality:** Optional
- **Content:** `@id` reference to a `schema:Identifier`. Identify this component definition in a global context for use in other documents.

### cdi:semantic

- **Cardinality:** Optional
- **Content:** array; each item is either a string IRI or a `cdifConceptOrTerm` (object or `@id` reference).
- **Description:** Qualifies the purpose or use of the component via an external controlled vocabulary.

## cdif:DimensionComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable that acts as a coordinate axis in a multidimensional structure. Used only in `cdi:DimensionalDataStructure`. Dimensions are typically categorical (codelist-valued) or quantized continuous variables (e.g., time bins).  The value domain for the represented variable associated with this component defines the dimension space.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:DimensionComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isDefinedBy_RepresentedVariable

- **Cardinality:** Required
- **Content:** `cdif:RepresentedVariable` inline or `@id` reference. Logical variable that contains values for this component.

## cdif:IdentifierComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable that identifies the unit (individual) that is the subject of properties specified in the record. Used in `cdi:WideDataStructure` and `cdi:LongDataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:IdentifierComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string, identifier for this node in the rdf graph.

### cdif:isDefinedBy_RepresentedVariable

- **Cardinality:** Required
- **Content:** `cdif:RepresentedVariable` inline or `@id` reference. Logical variable that contains values for this component.

## cdif:MeasureComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable that holds the observed or derived values of the dataset. Permitted in `cdi:DimensionalDataStructure` and `cdi:WideDataStructure`. (In `cdi:LongDataStructure` the measured value is carried by `cdif:VariableValueComponent` instead.)

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:MeasureComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isDefinedBy_RepresentedVariable

- **Cardinality:** Optional
- **Content:** `cdif:RepresentedVariable` inline or `@id` reference. Logical variable that contains values for this component.

### cdif:name

- **Cardinality:** Optional
- **Content:** array of string
- **Description:** Human-understandable name for the component. ISO/IEC 11179-5 naming principles may be followed.

### cdi:identifier

- **Cardinality:** Optional
- **Content:** `@id` reference to a `schema:Identifier`. Identify this component definition in a global context for use in other documents.

### cdi:semantic

- **Cardinality:** Optional
- **Content:** array; each item is either a string IRI or a `cdifConceptOrTerm`.

## cdif:VariableDescriptorComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable that holds codes identifying *which* logical variable a given row in a long-format dataset is recording. Used only in `cdi:LongDataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:VariableDescriptorComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isDefinedBy_DescriptorVariable

- **Cardinality:** Required
- **Content:** object — a `cdif:DescriptorVariable` whose values map to the logical variables of the dataset (see Data Description profile).

### cdi:refersTo

- **Cardinality:** Optional
- **Content:** `@id` reference to another `cdif:DataStructureComponent`.

### cdi:identifier

- **Cardinality:** Optional
- **Content:** `@id` reference to a `schema:Identifier`. Identify this component definition in a global context for use in other documents.

### cdi:semantic

- **Cardinality:** Optional
- **Content:** array; each item is either a string IRI or a `cdifConceptOrTerm`.

## cdif:VariableValueComponent

[↑ Back to TOC](#table-of-contents)

- Role given to a represented variable that carries the value of whichever logical variable the row's `VariableDescriptorComponent` identifies. Used only in `cdi:LongDataStructure`.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:VariableValueComponent`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isDefinedBy_RepresentedVariable

- **Cardinality:** Optional
- **Content:** `cdif:RepresentedVariable` inline or `@id` reference. Logical variable that contains values for this component.

### cdi:semantic

- **Cardinality:** Optional
- **Content:** array; each item is either a string IRI or a `cdifConceptOrTerm`.

## cdi:DimensionGroup

[↑ Back to TOC](#table-of-contents)

- A set of dimension components that together address a coordinate position within a `cdi:DimensionalDataStructure`. Used by structures where multiple dimensions share a notional axis (e.g., a `time` group containing year/month/day).

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdi:DimensionGroup`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdi:has_DimensionComponent

- **Cardinality:** Optional
- **Content:** array of `cdif:DimensionComponent` inline or `@id` references.

## cdif:PrimaryKey

[↑ Back to TOC](#table-of-contents)

- An ordered set of represented variables whose values uniquely identify a record in the dataset. Array order in `cdif:isComposedOf` is the key position; no intermediate ComponentPosition wrapper is used.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:PrimaryKey`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isComposedOf

- **Cardinality:** Required (≥1)
- **Content:** array of objects, each carrying a reference to a `cdif:RepresentedVariable` in the data structure and (implicitly via array order) the variable's position in the key.

## cdif:ForeignKey

[↑ Back to TOC](#table-of-contents)

- A set of represented variables whose values match a primary key in another dataset, expressing a cross-dataset reference.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:ForeignKey`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:isComposedOf

- **Cardinality:** Required (≥1)
- **Content:** array of objects referencing `cdif:RepresentedVariable`s in this data structure, in key order.

### cdi:references

- **Cardinality:** Required
- **Content:** `@id` reference to a `cdif:PrimaryKey` in a different dataset.

## cdif:RepresentedVariable

[↑ Back to TOC](#table-of-contents)

- A conceptual variable bound to a substantive value domain — *logical* in the sense that it is not tied to a particular physical data type or column position. Components in a DataStructure reference RepresentedVariables to indicate which logical variable plays which role. RepresentedVariables are the same thing across wide / long / dimensional structures, which is what lets the same dataset be presented in more than one layout.

### @type

- **Cardinality:** Required
- **Content:** array of string, contains `cdif:RepresentedVariable`.

### @id

- **Cardinality:** Optional
- **Content:** string. Identifier for this node in the rdf graph.

### cdif:name

- **Cardinality:** Optional
- **Content:** array of string

### cdif:displayLabel

- **Cardinality:** Optional
- **Content:** array of string (may be language-tagged)

### cdif:definition

- **Cardinality:** Optional
- **Content:** string
- **Description:** Natural-language meaning of the variable. Mutually exclusive with `cdi:externalDefinition`.

### cdi:externalDefinition

- **Cardinality:** Optional
- **Content:** object — reference to an external (e.g., SKOS) definition.

### cdif:descriptiveText

- **Cardinality:** Optional
- **Content:** string

### cdi:identifier

- **Cardinality:** Optional
- **Content:** `@id` reference to a `schema:Identifier`.  Identify this variable definition in a global context for use in other documents.

### cdi:hasIntendedDataType

- **Cardinality:** Optional
- **Content:** one of: string IRI, or `cdifConceptOrTerm` `@id` reference.
- **Description:** Intended physical datatype for variable values.

### cdi:describedUnitOfMeasure

- **Cardinality:** Optional
- **Content:** one of: string, `cdifConceptOrTerm` `@id` reference.

### cdi:simpleUnitOfMeasure

- **Cardinality:** Optional
- **Content:** string
- **Description:** Unit expressed as a simple string when no controlled-vocabulary entry is available or needed.

### cdi:unitOfMeasureKind

- **Cardinality:** Optional
- **Content:** one of: string, `cdifConceptOrTerm` `@id` reference.
- **Description:** Kind of unit (e.g., "temperature", "salinity"), allowing translation between equivalent units.

### cdi:measures

- **Cardinality:** Optional
- **Content:** inline object or `@id` reference to a `cdif:RepresentedVariable`.

### cdi:takesSubstantiveValuesFrom

- **Cardinality:** Optional
- **Content:** inline `cdi:SubstantiveValueDomain` or `@id` reference.
- **Description:** The set of valid, meaningful values for this variable.

### cdi:takesSentinelValuesFrom

- **Cardinality:** Optional
- **Content:** array of inline `cdi:SentinelValueDomain` or `@id` references.
- **Description:** Sentinel (missing / not-applicable) values for this variable.

### cdif:uses_Concept

- **Cardinality:** Optional
- **Content:** array of inline objects or `@id` references — concepts this variable expresses or aligns with.

## schema:Identifier

[↑ Back to TOC](#table-of-contents)

- A reusable identifier wrapper following the schema.org `schema:PropertyValue` pattern. **Union-type policy:** in CDIF profile UML models an attribute typed `schema:Identifier` / `schema:PropertyValue` is represented as a single attribute of that class. The JSON Schema permits the value to be EITHER a plain string (interpreted as the bare identifier value) OR a full `schema:PropertyValue` object (with explicit `@type`, `schema:propertyID`, `schema:value`). Consumers should accept both forms.

### @type

- **Cardinality:** Optional
- **Content:** `schema:PropertyValue`.

### schema:propertyID

- **Cardinality:** Optional
- **Content:** string
- **Description:** Identifier scheme (e.g., DOI, ARK). Values from <https://registry.identifiers.org/registry/> are recommended for interoperability.

### schema:value

- **Cardinality:** Optional
- **Content:** string
- **Description:** The identifier value, e.g., `10.5066/F7VX0DMQ`.

### schema:url

- **Cardinality:** Optional
- **Content:** string
- **Description:** Resolvable URL form of the identifier, e.g., `https://doi.org/10.5066/F7VX0DMQ`.

# Classes referenced from other CDIF profiles

[↑ Back to TOC](#table-of-contents)

These classes appear in the DataStructure UML model because the profile references them, but their authoritative definitions live in other profiles. The property tables below are copied here so this guide stands alone; the upstream definition is cited in each section header.

## dcat:CatalogRecord

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Core profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-core/blob/main/CDIFCoreImplementationGuide.md#dcatcatalogrecord).*

- The class used to provide information about the metadata record itself. Used here as the type of the `schema:subjectOf` node that carries the profile's `dcterms:conformsTo` declaration.

### @id

- **Cardinality:** Required
- **Content:** string.uri
- **Description:** Identifier for the metadata record.

### @type

- **Cardinality:** Required — `"Dataset"`, repeatable
- **Content:** string.uri

### schema:additionalType

- **Cardinality:** Required — `"dcat:CatalogRecord"`, repeatable
- **Content:** string

### schema:about

- **Cardinality:** Required
- **Content:** object reference
- **Description:** Reference to the metadata record's subject (the dataset) using the `@id` of that record.

### dcterms:conformsTo

- **Cardinality:** Required, repeatable
- **Content:** object reference
- **Description:** Identifiers for the conformance classes/profiles the metadata record follows. For the Data Structure profile this must include `https://w3id.org/cdif/data_structure/1.1`. Records that also conform to Core, Discovery, Data Description (etc.) list those profile identifiers as well.

### schema:description

- **Cardinality:** Optional
- **Content:** string
- **Description:** Other information about the metadata record.

### schema:maintainer

- **Cardinality:** Optional
- **Content:** Person or Organization
- **Description:** The agent that maintains the metadata, with contact information.

### sdDatePublished

- **Cardinality:** Optional
- **Content:** ISO 8601 date/datetime
- **Description:** Date of most recent update to the metadata content.

### schema:includedInDataCatalog

- **Cardinality:** Optional
- **Content:** DataCatalog
- **Description:** Identifies the source for the origin of the metadata record.

## schema:DataDownload

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Core profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-core/blob/main/CDIFCoreImplementationGuide.md#data-download).*

- File-based access to a resource via URL. The DataDownload object provides a link to the resource content along with information about the serialization format and conventions. The Data Structure profile adds `cdi:isStructuredBy` to this class (see [Properties added](#properties-added-to-schemadatadownload) above).

### @id

- **Cardinality:** Optional
- **Content:** string.uri
- **Description:** Node identifier; only necessary when the node will be referenced elsewhere in the graph.

### @type

- **Cardinality:** Required — `"schema:DataDownload"`, other types optional
- **Content:** string.uri

### schema:contentUrl

- **Cardinality:** Required
- **Content:** string.uri
- **Description:** HTTP URL that directly GETs the content of the resource described by this metadata record, in the format specified by `schema:encodingFormat`, conforming to the specifications identified in `dcterms:conformsTo`. Equivalent to `dcat:accessURL`. A landing page URL belongs in the dataset's `schema:url`, not here.

### schema:name

- **Cardinality:** Optional
- **Content:** string
- **Description:** Identifies this download option in user interfaces.

### schema:description

- **Cardinality:** Optional
- **Content:** string

### schema:encodingFormat

- **Cardinality:** Optional, repeatable
- **Content:** string (MIME type)
- **Description:** Identifier for format from a registry.

### spdx:checksum

- **Cardinality:** Optional
- **Content:** spdx:Checksum
- **Description:** Footprint of the described file enabling modification detection. The algorithm is specified by `spdx:algorithm`.

### dcterms:conformsTo

- **Cardinality:** Optional, repeatable
- **Content:** object reference
- **Description:** Identifier(s) for specification(s) the distribution conforms to. Recommended to enable machine-actionable data access.

### schema:provider

- **Cardinality:** Optional, repeatable
- **Content:** object reference, Person, or Organization
- **Description:** Agent responsible for access to the described resource.

### cdi:isStructuredBy

- See [Properties added](#cdiisstructuredby).

## schema:DefinedTerm

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Core profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-core/blob/main/CDIFCoreImplementationGuide.md#defined-term).*

- A term from a controlled vocabulary. Used as the value type for `cdi:semantic`, unit-of-measure, and similar properties when the term is supplied as a structured object rather than a bare IRI string.

### @type

- **Cardinality:** Required — `"DefinedTerm"`, repeatable
- **Content:** string.uri

### schema:name

- **Cardinality:** Required if no `identifier` or `termCode`
- **Content:** string
- **Description:** Label for the term.

### schema:identifier

- **Cardinality:** Required if no `name` or `termCode`
- **Content:** string.uri or PropertyValue-(identifier)

### schema:termCode

- **Cardinality:** Required if no `name` or `identifier`
- **Content:** string
- **Description:** Representative code for this term within the controlled vocabulary. Analogous to `skos:notation`.

### schema:inDefinedTermSet

- **Cardinality:** Optional
- **Content:** string
- **Description:** Name of the controlled vocabulary responsible for this term.

## cdif:UnitType

[↑ Back to TOC](#table-of-contents)

*Defined inline as part of the CDIF Data Structure Component vocabulary; a `schema:DefinedTerm`-style structured term used for unit-of-measure values that need machine semantics beyond a free-text label.*

- A type or class of objects of interest (units). Used here as a structured value of `cdi:measures` on a represented variable's unit description.

### @type

- **Cardinality:** Required, contains `cdi:UnitType`
- **Content:** array of string

### @id

- **Cardinality:** Optional
- **Content:** string

### cdif:displayLabel

- **Cardinality:** Optional
- **Content:** array of string (may be language-tagged)
- **Description:** Human-readable display label for the unit type.

### cdif:definition

- **Cardinality:** Optional
- **Content:** string
- **Description:** Natural-language definition. Mutually exclusive with `cdi:externalDefinition`.

### cdi:externalDefinition

- **Cardinality:** Optional
- **Content:** object — reference to an external (e.g., SKOS) definition.

### cdif:descriptiveText

- **Cardinality:** Optional
- **Content:** string

## cdifConceptOrTerm

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Data Description profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-datadescription/blob/main/CDIFDataDescriptionImplementationGuide.md#cdifconceptorterm).*

- A shared `$defs` type that accepts any of: an `@id` object reference, an inline `schema:DefinedTerm`, or a `skos:Concept` as defined in the CDIF Concept Scheme profile. Used wherever a property value can be either an external controlled-vocabulary concept or an inline definedTerm.

Used in this profile in `cdi:semantic`, `cdi:hasIntendedDataType`, `cdi:describedUnitOfMeasure`, `cdi:unitOfMeasureKind`, and similar slots.

## cdif:DescriptorVariable

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: `_sources/cdifDataType/cdifDescriptorVariable/` in [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks).*

- A presentational variable found only in long-format datasets. Its values reference the logical variables whose values populate the row — i.e., a `DescriptorVariable` answers "which represented variable's value is in this row's value column?" Referenced from `cdif:VariableDescriptorComponent.cdif:isDefinedBy_DescriptorVariable`.

### @type

- **Cardinality:** Required, contains `cdi:DescriptorVariable`
- **Content:** array of string

### @id

- **Cardinality:** Optional
- **Content:** string

### cdif:name

- **Cardinality:** Required (≥1)
- **Content:** array of string
- **Description:** Human-understandable name (linguistic signifier, word, phrase, or mnemonic). May follow ISO/IEC 11179-5 naming principles.

### cdif:hasValuesFrom

- **Cardinality:** Required
- **Content:** `cdif:DescriptorValueDomain` (inline)
- **Description:** Enumerates the codes that can appear in the descriptor column, each paired (via `cdif:isDefinedBy`) with the represented variable that the code names.

## cdif:DescriptorValueDomain

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: `_sources/cdifDataType/cdifDescriptorVariable/` (`$defs/DescriptorValueDomain`).*

- The set of permissible values for a variable playing the role of a variable descriptor component. Each entry pairs a code (`cdif:value`) with the represented variable that the code identifies (`cdif:isDefinedBy`).

### @type

- **Cardinality:** Required, contains `cdi:DescriptorValueDomain`
- **Content:** array of string

### @id

- **Cardinality:** Optional
- **Content:** string

### cdif:takesValuesFrom

- **Cardinality:** Required (≥1)
- **Content:** array of objects. Each item has:
    - **cdif:value** (required, string) — the code as it appears in the descriptor column.
    - **cdif:isDefinedBy** (required) — inline `cdif:RepresentedVariable` or `@id` reference to a represented variable defined elsewhere in the document.

## cdi:SubstantiveValueDomain

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Data Description profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-datadescription/blob/main/CDIFDataDescriptionImplementationGuide.md#cdisentinelvaluedomain-cdisubstantivevaluedomain).*

- The substantive (meaningful-value) domain of a `cdif:RepresentedVariable`. Differs from `cdi:SentinelValueDomain` only in the `@type` value: `cdi:SubstantiveValueDomain` vs. `cdi:SentinelValueDomain`. May be an enumerated codelist or a value space described via `cdif:ValueAndConceptDescription`.

### @type

- **Cardinality:** Required, contains `cdi:SubstantiveValueDomain`
- **Content:** array of string

### cdif:takesValuesFrom

- **Cardinality:** Optional
- **Content:** `cdif:EnumerationDomain` inline or `@id` reference.
- **Description:** Wrapper around a CDIF codelist that defines the enumerated values for this domain.

### cdif:displayLabel

- **Cardinality:** Optional
- **Content:** string
- **Description:** Human-readable display label for the value domain.

### cdif:recommendedDataType

- **Cardinality:** Optional
- **Content:** array of xsd data type strings.
- **Description:** Data type(s) recommended for use with this domain.

### cdi:isDescribedBy

- **Cardinality:** Optional
- **Content:** `cdif:ValueAndConceptDescription` inline or `@id` reference.
- **Description:** Formal description (ranges, patterns, classification level, expressions) of the values this domain admits.

## cdi:SentinelValueDomain

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Data Description profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-datadescription/blob/main/CDIFDataDescriptionImplementationGuide.md#cdisentinelvaluedomain-cdisubstantivevaluedomain).*

- The sentinel (missing / not-applicable / N/A code) value domain of a `cdif:RepresentedVariable`. Property set is the same as `cdi:SubstantiveValueDomain` (see above); only the `@type` differs (`cdi:SentinelValueDomain`). A represented variable may have multiple sentinel domains referenced from `cdi:takesSentinelValuesFrom`.

## cdif:EnumerationDomain

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Data Description profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-datadescription/blob/main/CDIFDataDescriptionImplementationGuide.md#cdifenumerationdomain).*

- A wrapper acting as an extension point that allows a CDIF Codelist to be documented as an enumerated value domain.

### cdif:identifier

- **Cardinality:** Optional
- **Content:** schema.org Identifier
- **Description:** Identifier for this enumeration domain.

### schema:name

- **Cardinality:** Optional
- **Content:** string
- **Description:** Label to identify this domain in user interfaces, if different from the underlying codelist's name.

### cdif:references

- **Cardinality:** Required
- **Content:** CDIF Codelist (`skos:ConceptScheme`) or `@id` reference to one.
- **Description:** The codelist whose notation values define the allowed values of this enumeration domain.

### cdif:purpose

- **Cardinality:** Optional
- **Content:** string
- **Description:** Intent or reason for the enumerated domain.

## CdifCodelistConcept

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Codelist profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-codelist/blob/main/CDIFCodelistImplementationGuide.md).*

- A SKOS Concept constrained for CDIF codelist use. Must have a resolvable `@id`, `skos:inScheme`, `skos:notation`, and `skos:prefLabel`. Because JSON-LD is an open-world implementation, any other SKOS properties may be included. Used here as the value type of the codelist concepts that populate a `cdif:EnumerationDomain` referenced from a `cdi:SubstantiveValueDomain`.

### @id

- **Cardinality:** Required
- **Content:** string
- **Description:** Globally unique, resolvable URI for this concept.

### skos:inScheme

- **Cardinality:** Required
- **Content:** array of object
- **Description:** The concept scheme this concept belongs to. Required for CDIF codelist concepts.

### skos:prefLabel

- **Cardinality:** Required
- **Content:** string
- **Description:** Preferred lexical label for this concept. A single string, a single language-tagged value, or an array of language-tagged values. Each language should appear at most once.

### skos:notation

- **Cardinality:** Required
- **Content:** string
- **Description:** Classification code for this concept within a scheme.

### skos:definition

- **Cardinality:** Optional
- **Content:** string
- **Description:** Formal definition of this concept. Optional for CDIF codelist concepts. A plain string.

### skos:narrower

- **Cardinality:** Optional
- **Content:** array of object
- **Description:** Narrower (child) concepts. If present, each inline concept must also declare `skos:broader` pointing back to the parent concept. Both `skos:narrower` and `skos:broader` must be explicit in CDIF codelists.

### skos:broader

- **Cardinality:** Optional
- **Content:** array of object
- **Description:** Broader (parent) concepts. Required on any concept that appears as a `skos:narrower` value of another concept. CDIF requires both directions to be explicit for hierarchy traversal.

## cdif:ValueAndConceptDescription

[↑ Back to TOC](#table-of-contents)

*Authoritative definition: [CDIF Data Description profile](https://github.com/Cross-Domain-Interoperability-Framework/profile-datadescription/blob/main/CDIFDataDescriptionImplementationGuide.md#cdifvalueandconceptdescription).*

- A formal description of a set of values — used when the substantive value domain is non-enumerated and must be characterized by patterns, ranges, expressions, or classification level rather than a discrete list.

### cdi:classificationLevel

- **Cardinality:** Optional
- **Content:** enumeration: `Continuous`, `Interval`, `Nominal`, `Ordinal`, `Ratio`.
- **Description:** Type of measurement-level relationship. Use where appropriate for the representation type.

### cdi:description

- **Cardinality:** Optional
- **Content:** string
- **Description:** Human-readable description of the value restrictions for this domain.

### cdi:formatPattern

- **Cardinality:** Optional
- **Content:** string
- **Description:** A pattern for numbers or dates as described in Unicode Locale Data Markup Language (LDML) Part 3 (Numbers) and Part 4 (Dates). Examples: `#,##0.###` for a decimal number; `yyyy.MM.ddTHH:mm:ss zzz` for a datetime.

### cdi:logicalExpression

- **Cardinality:** Optional
- **Content:** string
- **Description:** A logical expression where the values of `x` making the expression true are the members of the set of valid values. For example: `(all reals x such that x > 0)`.

### cdi:maximumValueExclusive

- **Cardinality:** Optional
- **Content:** string
- **Description:** Maximum valid value (exclusive). Per W3C "Metadata Vocabulary for Tabular Data" §5.11.2.

### cdi:maximumValueInclusive

- **Cardinality:** Optional
- **Content:** string
- **Description:** Maximum valid value (inclusive). Per W3C "Metadata Vocabulary for Tabular Data" §5.11.2.

### cdi:minimumValueExclusive

- **Cardinality:** Optional
- **Content:** string
- **Description:** Minimum valid value (exclusive).

### cdi:minimumValueInclusive

- **Cardinality:** Optional
- **Content:** string
- **Description:** Minimum valid value (inclusive).

### cdi:regularExpression

- **Cardinality:** Optional
- **Content:** string
- **Description:** A [regular expression](https://en.wikipedia.org/wiki/Regular_expression) that defines the valid syntax for value strings in this domain.

# Validation

[↑ Back to TOC](#table-of-contents)

Two validators ship with this repository:

- **JSON Schema** — `cdifDataStructureStructuredSchema.json` (Draft 2020-12), generated from the source register.
- **SHACL** — `dataStructureRules.shacl`, a self-contained shapes graph merged from every composing building block plus the profile-level shapes.

```bash
python FrameAndValidate.py examples/<file>.json --validate \
  --schema cdifDataStructureStructuredSchema.json \
  --frame CDIFDataStructure-frame.jsonld
```

Validation is **open-world**: properties not described by the profile are allowed.

# Provenance of the artifacts

[↑ Back to TOC](#table-of-contents)

The schema and SHACL files are generated from the canonical source register, [metadataBuildingBlocks](https://github.com/Cross-Domain-Interoperability-Framework/metadataBuildingBlocks):

- `cdifDataStructureStructuredSchema.json` ← `tools/resolve_schema.py cdifDataStructure`
- `dataStructureRules.shacl` ← `tools/validate_shacl.py cdifDataStructure --emit-shapes`

Source profile directory: `_sources/profiles/cdifProfile/cdifDataStructure/`.
