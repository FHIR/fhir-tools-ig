The IG publisher can be configured to produce a page for each item in a list. 
There can be multiple page factories. Each page factory is registered as a 
parameter using the ```page-factory``` parameter.

Each page factory is a refernece to a page factory control file which is a json file 
with the following format:

```json
{
  "item-factory" : "{code}",
  "source-file" : "{path}",
  "variables" : [{
    "name" : "{code}",
    "transform" : "{code}"
  }],  
  "generated-file-name": "{filename}",
  "stated-file-name": "{filename}",
  "parent-page" : "{filename}",
  "generation" : "{code}",
  "page-title" : "{string}"
}
```

For each item in the factory list, an output file will be produced.

## Documentation:

* **item-factory**: The kind of item being iterated. See below for possible factory values
* **source-file**: the name of the file (relative path in the repository) for the source to produce
* **variables**: (optional) a list of variables to replace in the source file. See below.
the value of a string to replace in the content of the source file. Any occurrences of this string value with be replaced with the item value. If this is not present, no search/replace will take place
* **generated-file-name**: the name of the file to produce in the page directory. The filename will include %item%, and this will be replaced with the item value
* **stated-file-name**: the name of the file to put in the IG (not necessarily the same as the generated file name - e.g. may have a different extension). The filename will include %item%, and this will be replaced with the item value
* **parent-page**: the name of the page to which the generated pages will be added as child pages in the ToC
* **generation**: THe value to put in the generation entry for the page
* **page-title**:  the title for the page file in the ToC. The filename may include %item%, and this will be replaced with the item value

## Possible Factory types:

* **types**: a list of all types defined in FHIR
* **resources**: a list of all resources defined in FHIR
* **datatypes**: a list of all datatypes defined in FHIR
* **canonicals**: a list of all canonical resources defined in FHIR
* **versions**: A list of all the official release versions

## Variables

A list of variables to replace in the source file. Each variable
has a name and a transform. The page factory performs the transform
on the item being iterated, and than replaces all occurences of 
the text string given in the name with the trassformed value.

Known transforms:

* **n/a**: No transform - use the value as is
* **lowercase**: Convert the item to all lower case 
* **uppercase**: Convert the item to all upper case 
* **version-title**: For versions, convert to R2 / R3 / R4 etc 
* **tla**: For resource names, look up the known tla for the version

