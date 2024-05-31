# Tooling IG

This documents the profiles, extensions, value sets, and code systems 
defined by the HL7 tooling. Principally, this is the Java validator and 
the IG publisher, though these resources are used by other tools as well.
This IG is used when building all IGs, and is always in scope for internal 
tooling reasons. There is no expectation that these resources are applicable to real 
world healthcare systems.

Latest version is available at [https://build.fhir.org/ig/FHIR/fhir-tools-ig].

## FHIR Foundation Project Statement

* Maintainers: Grahame Grieve (as FHIR Product Director) + Lloyd Mackenzie
* Issues / Discussion: Create Issues here on GitHub. Discussion about (potential) issues, see [Zulip](https://chat.fhir.org/#narrow/stream/179239-tooling). Note that issues associated with the Tooling IG are generally issues with the tools as a whole
* License: [CCO - Creative Commons Public Domain](https://github.com/FHIR/fhir-tools-ig/blob/master/LICENSE.txt)
* Contribution Policy: Contributions to this IG are made under [the HL7 GOM](https://www.hl7.org/permalink/?GOM). This IG uses the standard HL7 ci-build for IGs, checking the generated QA. The content is reviewed by the [FHIR-I](https://www.hl7.org/Special/committees/fiwg/index.cfm) committee prior to publication
* Security Information: [see below](#security)
* Compliance Information: This content is used by/integrated with the base FHIR tools and is used across all versions of FHIR (R2-R5)

## Security

As an implementation guide that includes no active content, there's no direct security related content. 

For issues with the scripts that launch the publisher, see https://raw.githubusercontent.com/HL7/ig-publisher-scripts

If you think that there's security issues in the tools that implement these extensions etc, you can report them 
either on GitHub ([Validator](https://github.com/hapifhir/org.hl7.fhir.core), 
[IG Publisher](https://github.com/HL7/fhir-ig-publisher), [FHIRServer](https://github.com/HealthIntersections/fhirserver)) 
using GitHub's [standard security reporting framework](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing-information-about-vulnerabilities/privately-reporting-a-security-vulnerability#privately-reporting-a-security-vulnerability), or you can email the [FHIR Director](mailto:fhir-director@hl7.org) directly.

If you think that the content of this IG has designs that imply bad security decisions, feel free to raise 
an issue using on GitHub, or discuss on [Zulip](https://chat.fhir.org/#narrow/stream/179239-tooling). 
