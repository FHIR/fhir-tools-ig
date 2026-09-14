
The way that a content model is represented in JSON or XML is fixed for FHIR resources 
and data types. But when logical models are defined that represent existing JSON 
and XML formats, the FHIR rules do not apply. The extensions documented on this page 
define alternative rules that apply to logical models. They are never used in FHIR
resources.

### Type related Extensions

These extensions control how the type is chosen when there's an choice of types.

For FHIR resources, the type of a choice element is indicated by fixing the name of the 
type to the element name. So e.g. if the element name is ```value[x]```, then the name 
of the element will be ```valueString``` if it has the type of string. 

There are two other ways to do this: by xsi:type and by a type specifier.

xsi:type is specified by the ElementDefinition.representation value of ```typeAttr```.
If this is set, the type will be determined by xsi:type. If there's a default type
then this can be specified by the 
```http://hl7.org/fhir/StructureDefinition/structuredefinition-explicit-type-name```
extension.

Alternatively, in some JSON files, there's no property that defines the type; instead,
the type is determined by the property of some other value. The extension 
```http://hl7.org/fhir/tools/StructureDefinition/type-specifier``` exists for this purpose.
It has two sub-extensions, an expression and a type.

The expression is a FHIRPath expression that determines what the type is, that is 
evaluated from the root of the JSON object being parsed. This is tricky, since the 
resource is still being parsed at this point, so the FHIRPath expression must be 
confined to the parts of the resource already parsed (by the logical model definition).

The type is a canonical URL reference to a logical model. If there is more than one 
type specifier, they are evaluated in order and the first match is used.

### Extension related Extensions

These extensions control how and when elements can be extended. The base FHIR extension 
model is that the type Element defines an extension that has a pair of properties, a 
url which references a definition, and a value. Then this url/value pair is constrained
to define particular extensions in applicable profiles, or an application just provides 
the url/value pair directly. Logical Models that inherit from the Element type inherit
this extension mechanism.

The Element type inherits from Base. Logical Models that inherit from Base do not have 
any inherent extensions. Note that the type Base is implicitly defined in FHIR R4 and 
is made explicit in R5. 

The FHIR Extension ```http://hl7.org/fhir/tools/StructureDefinition/extension-style```
defines how extensions are managed. In the absence of the extension, there are no extensions.
If the extension is present, it may have one of the following values:

* **none**: No extensions
* **fhir-extensions**: FHIR extension style: the element defines 'extension' which can be constrained by profiles
* **named-elements**: The type can be extended by having additional named elements, where the name is from the ```xml-name``` or ```json-name``` extension for xml and json respectively

### JSON Extensions

These elements control the JSON format. They're mainly used for describing CDSHooks format using StructureDefinitions.

```http://hl7.org/fhir/tools/StructureDefinition/json-name```

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

If an array property is empty, then it must normally be omitted. This extension
can modify this rule to specify that the array may or must be present and empty instead of omitted.

```http://hl7.org/fhir/tools/StructureDefinition/json-nullable```

If an object property is null, then it must normally be omitted. This extension
can modify this rule to specify that the object may or must be present as null instead of omitted.

```http://hl7.org/fhir/tools/StructureDefinition/json-primitive-choice```

Marks an element as a choice of types where the type is not named in the property 
name (as it is for FHIR resources), but is implied by the JSON type of the value. 
Only ```string```, ```integer```, ```decimal``` and ```boolean``` can be chosen 
between this way, since those are the only types JSON itself distinguishes.

```http://hl7.org/fhir/tools/StructureDefinition/json-suppress-resourcetype```

By default, the JSON format produced from a logical model includes a 
```resourceType``` property naming the model, and that property is used to 
recognise the format when reading. If this extension is present and true, the 
```resourceType``` property is not written, and the content is not recognised 
automatically when read - the reader must be told what the content is. This 
extension is used on the StructureDefinition, not on an element.

### XML Extensions

These elements control the XML format. They're mainly used to define the CDA format using StructureDefinitions

```http://hl7.org/fhir/StructureDefinition/structuredefinition-xml-type```

The schema type associated with the FHIR primitive type

```http://hl7.org/fhir/tools/StructureDefinition/xml-name```

The name of the xml element in the instance. This has two uses: if the actual xml name is not 
a legal ElementDefinition path name (e.g. includes '.'), or if there is a need to define a name for the type itself,
for use with ```named-elements``` - see above.

```http://hl7.org/fhir/tools/StructureDefinition/xml-namespace```

The XML namespace for the element, where it differs from ```http://hl7.org/fhir```. 
The special value ```noNamespace``` indicates that the element has no XML namespace 
at all.

```http://hl7.org/fhir/tools/StructureDefinition/xml-no-order```

Elements must appear in the XML in the order they are defined in a logical model.  
This extension can modify this rule to specify that elements can appear in any 
order.

```http://hl7.org/fhir/tools/StructureDefinition/xml-choice-group```

If true, marks the element as a choice group that does not literally appear in the 
XML at all; it exists only to group a set of repeating elements. The children of the 
element appear in the XML in place of the element itself.

### Date/time Formats

```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-date-format```

The default date/time format in all formats (XML, JSON, RDF) is the XML schema 
date/time type. Logical models can use this extension to specify an alternative 
date foramt. The following values are legal:

* YYYYMMDDHHMMSS.UUUU[+|-ZZzz] (v2/v3 date format)

### Date/time Validation Rules

```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-date-rules```

A set of colon delimited codes that control which validation rules are enforced on 
date and date/time elements. If the extension is not present, all the rules are 
enforced; if it is present, only the rules named in it are. The codes are:

* **tz-for-time**: enforce the rule that there must be a timezone if there is a time
* **year-valid**: enforce the rule that the year is in the range 1800 -> now + 80 years

This extension has no effect on data types and resources - only on logical models.

### String Formats

```http://hl7.org/fhir/tools/StructureDefinition/elementdefinition-string-format```

The actual format of a string element, so that it can be validated further. This 
usually arises with layered content models, where an outer model carries a value as 
a string that the inner model knows to be something more specific. The value of the 
extension is the name of another primitive type, and the string is validated as if 
it had that type.

```http://hl7.org/fhir/tools/StructureDefinition/implied-string-prefix```

A prefix that is automatically prepended to the value before it is validated. This is 
for use in logical models where the instance carries only part of a value - the local 
part of an identifier, say - and the rest is implied by the model rather than stated 
in the instance.
