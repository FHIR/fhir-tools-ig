
# Parameterised ValueSets V0.2

## Background and Motivation

A common activity in a data entry scenario like a Questionnaire is to have the set of available options for one question depend on an answer to another question.
For example, after choosing a country, the set of available options for a question about a state or region would depend on the chosen country.
Ideally a ValueSet would be bound to the state/region question, but since it needs to be pre-defined we cannot have one that is sensitive to the country.

Similar examples exist when questions correlating diagnoses or procedures and body sites/regions, or medications, or tests.

Parameterised value sets can be used in these situations. Setting up parameterised value sets has 3 steps:

* Declare the parameter to the value set using the [ValueSet Parameter Declaration Extension](StructureDefinition-valueset-parameter.html). This names the parameter, and provides documentation
* Use the parameter in the definition of the value set to narrow the possible set of values returned by $expand or allowed by $validate-code (using the [CQF Expression extension](https://hl7.org/fhir/extensions/StructureDefinition-cqf-expression.html))
* Define the values for the parameter using [Binding Parameter Declaration Extension](StructureDefinition-binding-parameter.html). This also names the parameter, and provides an expression that provides the value for the parameter

The value set author does the first two steps. The value set author does the last step when binding the value set to their content. Parameterised ValueSets 
can be used in questionnaires or profiles, though the profile usage is as yet unexplored.

### Parameterised ValueSet definition

Parameterised ValueSets are defined in the same way as regular ValueSets, except that:
1. the [ValueSet Parameter Declaration](StructureDefinition-valueset-parameter.html) extension is used to define and document the parameter names, and
2. the `value` field of the `ValueSet.compose.include.filter[i]` is left empty, and the [CQF Expression extension](https://hl7.org/fhir/extensions/StructureDefinition-cqf-expression.html) is used to supply its value.

```json
{
  "resourceType": "ValueSet",
  "extension" : [{
    "url": "http://hl7.org/fhir/tools/StructureDefinition/valueset-parameter",
    "extension" : [{
      "url": "name",
      "valueCode" : "p-inactive",
    }, {
      "url": "documentation",
      "valueMarkdown" : "whether the record is inactive or not"
    }]
  }],
  "url": "http://example.com/ValueSet/parameterised",
  "compose": {
    "include": [{
      "valueSet": [ "http://snomed.info/sct?fhir_vs=refset/929360041000036105" ],
      "system": "http://snomed.info/sct",
      "filter": [{
        "property": "inactive",
        "op": "=",
        "_value": {
          "extension" :[{
            "url" : "http://hl7.org/fhir/StructureDefinition/cqf-expression",
            "valueExpression" : {
              "language" : "text/fhirpath",
              "expression" : "%p-inactive"
            }
          }]
        }
      }]
    }]
  }
}
```

#### Notes:
* The expression SHALL be a FHIRPath expression, using a restricted subset of the [FHIRPath](http://hl7.org/fhir/fhirpath.html) specification. The following functions SHALL not be used, and servers do not make them available:
  * `resolve()`
  * `elementDefinition()`
  * `slice()`
  * conformsTo()
  * any functions defined for the Type Factory and the General Service API
  * Note that the terminology API is expected to be available
* All the parameters passed to the $expand or $validate-code operation are made available to the expression as FHIRPath constants (no matter how the parameter was passed)
* Separation of "definition" and "use" enables the parameter to be used more than once in different parts of the `ValueSet.compose` and provides a place to document the parameter.
* Using `cqf-expression` rather than just naming the parameter allows for additional logic to be applied to the parameter value. For example, the parameter value might be one of "left" or "right", and then it can be mapped to the corresponding SNOMED CT code in one filter and the corresponding LOINC code in another filter.
* While it is possible to define arbitrary parameter names, care should be taken to avoid naming conflicts with existing parameters to `ValueSet/$expand` and `ValueSet/$validate-code`.
* parameter names SHOULD start with  `p-` to avoid naming conflicts, and for ease of processing on API Gateways and servers

### Parameterised ValueSet expansion

Having defined a parameterised ValueSet, you can expand it by providing the values for the parameters as query parameters in a GET request or in as `Parameters.parameter` elements in a POST request.

The following GET request:
`/ValueSet/$expand?url=http://example.com/ValueSet/parameterised&p-inactive=true`
will result in the expansion of a ValueSet defined as follows:
```json
{
  "resourceType": "ValueSet",
  "url": "http://example.com/ValueSet/parameterised",
  "compose": {
    "include": [{
      "valueSet": [ "http://snomed.info/sct?fhir_vs=refset/929360041000036105" ],
      "system": "http://snomed.info/sct",
      "filter": [{
        "property": "inactive",
        "op": "=",
        "value": "true"
      }],
    }]
  }
}
```

All parameters that are used in the expansion SHALL be included in the `ValueSet.expansion.parameters` element of the resulting ValueSet.

If the cqf-expression evaluates to `()`, then corresponding filter element is omitted from the compose definition used for expansion.
Hence the following GET request:
`/ValueSet/$expand?url=http://example.com/ValueSet/parameterised`
will result in the expansion of a ValueSet defined as follows:
```json
{
  "resourceType": "ValueSet",
  "url": "http://example.com/ValueSet/parameterised",
  "compose": {
    "include": [{
      "valueSet": [ "http://snomed.info/sct?fhir_vs=refset/929360041000036105" ],
      "system": "http://snomed.info/sct",
    }]
  }
}
```

It is the terminology server's discretion to decide whether the parameterised ValueSet.compose or the computed ValueSet.compose is used in the response when for `includeDefinition=true`.


## Use in Questionnaires

A primary use context for parameterised ValueSets is in Questionnaires, using the  [Binding Parameter Declaration Extension](StructureDefinition-binding-parameter.html).

```json
{
  "linkId": "language_vsc.3",
  "text": "language (valueset canonical - no version)",
  "type": "choice",
  "_answerValueSet":{
    "extension": [{
      "url": "http://hl7.org/fhir/tools/StructureDefinition/binding-parameter",
      "extension": [{
        "name":"name",
        "valueCode": "p-inactive"            
      },{
        "name":"expression",
        "valueExpression": {
            "language" : "text/fhirpath",
            "expression":"item(12).answer.valueString.join(',')"              
        }
      }
    },{
      "url": "http://hl7.org/fhir/tools/StructureDefinition/binding-parameter",
      "extension": [{
        "name":"name",
        "valueCode": "language"            
      },{
        "name":"expression",
        "valueString": "en-AU"
      }
    }]
  },
  "answerValueSet": "http://hl7.org/fhir/ValueSet/languages"
}
```

Notes:
* WHen an expression is provided, it is evaluated under the rules defined in the [SDC specification](https://build.fhir.org/ig/HL7/sdc/behavior.html)

## Use in Profiles

TBD
