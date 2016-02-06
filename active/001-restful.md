---
Feature name: restful
Start date: TBD
Pull request: <leave empty>
Authors: ["@wasade"]
Contributors:
---

# Summary

Provide a RESTful interface into the Greengenes database provideing an API to enable: community comment on records, benchmarking methods, and provenance tracking of artifacts.  

# Motivation

The primary challenge with Greengenes has been a lack of central infrastructure. Each release of Greengenes generates a large number of artifacts that have rich relational structure within and between releases. A lack of a centralized resource makes formally representing these relations and any community-added details about the artifacts impossible. In addition, each release requires iteration at a few steps, particularly during the curation phase as the filtering criteria on the sequence records is insufficient to capture all of the bad sequences that make it into the tree. Since Greengenes relies on a de novo phylogeny, it is particularly sensitive to poor quality sequence.

The development of a central resource with a RESTful interface supports, at a minimum, the following use cases:

* independent and automated workers for tasks (e.g. searching for records of interest)
* provenance tracking of artifacts (e.g., sequence X was aligned with method FOO and is part of phylogeny BAR)
* crowd sourcing of curation from the community
* exploration of methods that operate on the data
* standardized iteration cycles for development and production releases

The expected outcome of this RFC is a RESTful API for community and development use that resides on the public internet in a single stable location. Following the definition of this API, definition of automated workers will become the focus. 

## Present way a release is constructed

The current method for generating a Greengenes release first requires broad queries against NCBI to find records that tentatively contain 16S rRNA genes. These queries are filtered to remove any records that appear to have been previously retrieved by Greengenes. The records are then processed to determine whether the record appears to be associated with a clone or isolate, strain details and author-provided taxonomy. The sequences are then aligned by SSU-Align in order to identify 16S rRNA genes. Once the genes have been identified, their sequences are then re-aligned by PyNAST to assist in subsequent curation efforts, and to assess the percentage of positions that appear to vary relative to positions in the core set that are invariant. The SSU-Align'd records are then masked by SSU-Align. Sequence length and percentage of non-ATGC characters are tracked. 

All alignments, and the metrics (e.g., percentage of non-ATGC) are then inserted into the present Greengenes database. The set of tentative new sequences is then defined as those not in prior releases, and are >= 1200nt, >= 90% invariant relative to the core set and < 1% non-ATGC. Previous curation of releases has highlighed that sequences derived from the EMIRGE method are universally bad in terms of their base calls, and so any record that indicates it is EMIRGE is removed. The set of tentative sequences is then subjected to chimera detection first by performing closed reference OTU picking at 99% identity against previously identified chimeras. The records are additionally checked using UCHIME with the previous releases 99% representative members as a reference; any new sequence that appears to bridge taxonomic classes is marked as chimeric. A release candidate set of records is then defined as all of the records from the existing release (minus any that may have been flagged as bad between releases), and all new sequences that pass the defined quality metrics and chimera filtering. 

One a tentative sequence set is defined, a reduction of the sequence data is performed in order to reduce curation burden. The full release of Greengenes as of May 2013 was over 1.2 million records, and representing that phylogeny in ARB makes curation difficult. In order to make it easier to compare old releases to release candidates, it is important that, where feasible, the representative sequences used in the reduced set overlap with the existing release. One strategy is to perform a closed-reference OTU picking against the prior releases' 99% representative members. Any sequences that failed to recruit can then be de novo OTU picked, however, representative choice matters as it is more informative to have named isolates on the tips of the tree for curation than for clone members. As such, sequences that fail to recruit to the prior release are sorted by whether they are an isolate or clone, then a secondary sort is performed based on the sequence length. The sort assumes of course that the OTU picking method will preferentially use earlier sequences as centroids. 

Following determination of a reduced set of sequences, the SSU-Align'd and Lane masked records are then run through FastTree to reconstruct a phylogeny. This phylogeny is then rooted based on the prior releases knowledge of archaea and bacteria. Following rooting, the prior releases taxonomy is decorated onto the tree using tax2tree. The tree with the decorated taxonomy, and a dump of the new records in ARB format, is then sent off to Phil Hugenholtz for a curation cycle. The curation cycle will generally identify additional chimeras and poor quality sequence, and a judgement call is made as to whether the records should be filtered out and a new tree is constructed -- this is a manual effort requiring communication between both Phil and Daniel to generate the new trees. 

## How infrastructure improves releases

Many of the steps to developing a release can be automated and continuous. For instance, the identification of new records to include can is something that can be performed by automated workers, and on a continuous basis. Similarly, the production of alignments under a variety of methods, and the assessment of quality of those artifacts is an easy and direct target for automation. But automation is not possible without a way to track provenance of the records. These automation components allow for a broad and consistent growth of the resource with minimal human involvement. 

The extent of automation can be taken all the way up to the manual curation phase as through the use of periodic workers that define sequence sets based off of filtering criteria. Ultimately, these are just relations tracked in a database. Drawing the concept of automated components out further, it becomes possible to have a "development" release of Greengenes that is automatically produced on a stable interation cycle and of course, having defined and expected releases on a broader cycling interval.

A key concept in the potential for automation is that it is independent of the API. 

## How infrastructure facilitates benchmarking

Greengenes right now is only composed of 16S rRNA. The limiting factor for growing to multiple loci is human effort to explore different methods and their impact for this expansion. Even within 16S rRNA, the approaches to how sequences are aligned and QC'd is an open question (and without a doubt, there are better things that can be done). The limiting factor here, as well, is human effort. 

By building off of the potential for automation that the infrastructure enables, it becomes easier to define methods to execute that can just become part of the regular automated workforce. Fundamentally, the methods that operate on the data are independent of the API.  

## How infrastructure supports the community 

A defined API makes it easier to build a separate and independent user interface. A minimal interface could be composed of ways to retrieve information about a given record by either its GG ID or its INSDC accession. Behind the API, since the relations are tracked, it becomes possible to present a rich set of information to the user such as what alignments are available, what releases a record is present in, quality metrics, and multiple taxonomies (e.g., old releases, NCBI, SILVA, etc). 

Perhaps more useful going forward, it becomes possible to allow users to make curation recommendations. The list of taxonomies presented to the user can include community recommended changes. Once we begin crowd sourcing recommendations, collating those for a curator becomes a (relatively) straight forward task.
 
# Detailed design

Actual details on the API go here.

# Drawbacks

It takes a lot of effort.

# Alternatives

* We let Greengenes die.
* We continue with manual efforts.
* We coerce Greengenes into UNITE.

The impact of not doing this is that the microbiome community will fully shift away from Greengenes. In the candidate parts of the tree of life, this may encourage the development of a large number of small reference databases which are unlikely to be consistent with each other. This will add noise to the literature.  

Greengenes provides a fundamentally unique perspective on microbial life, both thanks to its expert curation and its reliance on de novo phylogenies to represent the diversity of the candidate parts of the tree. As the microbiome field is under rapid expansion and exploration, providing insight into the "dark" parts of the tree is becoming of increasing value. 

# Unresolved questions

What parts of the design still need to be fleshed out? This may be left blank.
