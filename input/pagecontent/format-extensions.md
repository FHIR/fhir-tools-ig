
The way that a content model is represented in JSON or XML is fixed for FHIR resources 
and data types. But when logical models are defined that represent existing JSON 
and XML formats, the FHIR rules do not apply. The extensions documented on this page 
define alternative rules that apply to logical models. They are never used in FHIR
resources.

### Type related Extensions

These extensions control how the type is chosen when there's an choice of types.

For FHIR resources, the type of a choice element is indicated by fixing the name of the 
type to the element name. So e.g. if the element name is ```value[x]``, then the name 
of the element with be ```valueString``` if it has the type of string. 

There are two other ways to do this: by xsi:type and by a type specifier.

xsi:type is specified by the ElementDefinition.representation value of ```typeAttr```.
If this is set, the type will be determined by xsi:type. If there's a default type
then this can be specified by the 
```http://hl7.org/fhir/StructureDefinition/structuredefinition-explicit-type-name```
extension.

Alternatively, in some JSON files, there's no property that defines the type; instead,
the type is determined by the property of some other value. The extension 
```http://hl7.org/fhir/tools/StructureDefinition/type-specifier``` exists for this purpose.
it has two sub-extensions, an expression and a type.

The expression is a FHIRPath expression that determines what the type is, that is 
evaluated from the root of the JSON object being parsed. This is tricky, since the 
resource is stil being parsed at this point, so the FHIRPath expression must be 
confined to the parts of the resource already parsed (by the logical model definition).

The type is a canonical URL reference to a logical model. If there is more than one 
type specifier, they are evaluated in order and the first match is used.

### Extension related Extensions

The extensions control how and when elements can be extended. The base FHIR extension 
model is that the type Element defines an extension that has a pair of properties, a 
url which references a definition, and a value. Then this url/value pair is constrained
to define particular extensions in applicable profiles, or an application just provides 
the url/value pair directly. Logical Models that inherit from the Element type inherit
this extension mechanism.

The Element type inherits from Base. Logical Models that inherit from Base do not have 
any inherent extensions. Note that the type Base is implicitly defined in FHIR R4 and 
is made explicit in R5. 

The FHIR Extension ```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-extension-style```
defines how extensions are managed. In the absence of the extension, there are no extensions.
If the extension is present, it may have one of the following values:

* **none**: No extensions
* **fhir-extensions**: fhir-extensions
* **named-elements**: The type can be extended by having additional named elements, where the name is from the ```elementdefinition-xml-name``` or ```elementdefinition-json-name``` for xml and json respectively"

### JSON Extensions

These elements control the JSON format. They're mainly used for describing CDSHooks format using StructureDefinitions.

```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-json-name```

The name of the json property in the instance. This has two uses: if the actual json name is not 
a legal ElementDefinition path name, or if there is a need to define a name for the type itself,
for use with ```named-elements``` - see above.

```http://hl7.org/fhir/tools/StructureDefinition/json-property-key```:

if a property key is specified on a repeating JSON element, then the element 
is represented as a JSON object rather than a JSON array, and there will be 
a JSON property for each entry in the array with a name which is the value of 
the key property extension. 

The property-key approach can only be used when there is an element that repeats,
which has two properties, a key and a value (though they don't have to be called that).
The key must be a primitive type (typically a ```code```) that can't have extensions.
The value sub-element can have any type.

```http://hl7.org/fhir/tools/StructureDefinition/json-empty-behavior```

if an array property is empty, then it most normally be omitted. This extension
can modify this rule to specify that the array may or must be present and empty instead of omitted.

```http://hl7.org/fhir/tools/StructureDefinition/json-nullable```

if an object property is null, then it most normally be omitted. This extension
can modify this rule to specify that the object may or must be present a null instead of omitted.

### XML Extensions

These elements control the XML format. They're mainly used to define the CDA format using StructureDefinitions

```http://hl7.org/fhir/StructureDefinition/structuredefinition-xml-type```

The schema type associated with the FHIR primitive type

```http://hl7.org/fhir/StructureDefinition/elementdefinition-xml-name```

The name of the sml element in the instance. This has two uses: if the actual xml name is not 
a legal ElementDefinition path name (e.g. includes '.'), or if there is a need to define a name for the type itself,
for use with ```named-elements``` - see above.

```http://hl7.org/fhir/StructureDefinition/elementdefinition-namespace```

The XML namespace for the element 

```http://hl7.org/fhir/StructureDefinition/structuredefinition-xml-no-order```

Elements must appear in the XML in the order they are defined in a logical model.  
This extension can modify this rule to specify that elements can appear in any 
order.

### Date/time Formats

```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-date-format```

The default date/time format in all formats (XML, JSON, RDF) is the XML schema 
date/time type. Logical models can use this extension to specify an alternative 
date foramt. The following values are legal:

* YYYYMMDDHHMMSS.UUUU[+|-ZZzz] (v2/v3 date format)
