# OpenContracting

## Background

OpenContracting is an effort to standardise how contracts are published. The aims are:

  * for more transparency in government spending
  * for better comparability across contracts

> This review paper looks in general terms at the technical architecture for a standard such as OpenContracting. It does not recommend specific vocabulary or data structures, only general patterns and approaches. Examples are given purely to ground the discussion and make it more understandable, not to constrain the details of the standard.

## Technology

Contracts and information about contracts are already published in many countries, in many formats. For example:

  * TODO: HTML example
  * TODO: XML example
  * TODO: CSV example

It is useful to distinguish between the contracts themselves, which are narrative documents, and useful information about contracts (metadata).

### Representing Metadata

Most of the analysis of open contracting data needs to be carried out on metadata such as:

  * who the contract is between
  * the total amount of the contract
  * the term of the contract
  * the location of the published contract itself

Publication of metadata may be limited to a single contract, but in many cases the details of many contracts will be published at once.

Data formats are standards for passing data between applications. Once data is imported into an application, it is processed using internal structures, usually object structures, which may bear little relationship with the format that is used for sharing data between applications. It is rare for the format of some data to determine the structures used within a program. Every application needs to interpret the data into its internal structures. A written specification (and reference implementation) is used by developers to understand how the data should be interpreted.

Standards can invent completely new syntaxes for encoding data. However, there are also several common meta-languages such as XML and JSON. These meta-languages specify some basic parsing rules. Using a common meta-language reduces the burden on developers who are importing or exporting data. It also means that common standard query languages can be used on the data by generic processors. For this reason, new standards are commonly built on top of these meta-languages.

The meta-languages that most standards use are:

  * XML
  * JSON
  * CSV
  * RDF (and formats related to RDF)

The commonly cited reasons for using each of these meta-languages are:

  * XML is well-supported across platforms, good at representing mixed, typed and annotated content, but very verbose and doesn't fit with most developer's mental models for data
  * JSON is also well-supported across platforms, a little less verbose, fits well with programming languages, but has a limited range of types
  * CSV is extremely compact for lots of data with essentially the same structure, but doesn't deal well with repeating or mixed content and has no mechanisms for representing metadata[^1]
  * RDF comes in a range of formats (RDF/XML, JSON-LD, Turtle), good at representing typed content and links, but its graph model doesn't match with most developers' understanding of data

[^1]: See the work of the CSV on the Web Working Group

However, there is always a way to represent a given data structure in any of these meta-languages. For example, JSON doesn't have a native way of indicating the language used within a document or an individual value, but if it was necessary, a format based on JSON could specify the language for the document in a top-level metadata section, or use objects with `lang` and `value` properties to represent such values.

From experience with other data standards, **it is very unlikely that a format based on any individual meta-language will suit all publishers or all developers**. Moreover, it is likely that if a popular standard is created that is built on, say, XML, then some web developers will create mappings to JSON so that they can use it in browsers, some researchers will create a mapping to RDF so that they can link it to other data, and some publishers will publish using CSV because it's the easiest thing for them to produce.

If alternative mappings are designed by individual developers, there's a strong likelihood that they will not all choose the same method of mapping from the standard format into the formats they prefer. This could lead to an effective balkanisation, even if there is a single standard format in place.

> NOTE: Add example?

For this reason, we recommend:

  1. specifying a single format-independent underlying data model and vocabulary
  2. specifying how this data model should be represented in the key formats
      * XML
      * JSON
      * CSV
      * RDF
  3. if the data is to be hand-authored, also specifying a simple text-based format

An important implementation in this model is a processor that can parse data in any of these formats into an internal data model, and then output that same data model as any of these formats.

#### Whether to use RDF

Aside from a few gotchas, such as its representations of lists, RDF is a good generic data model for web-based metadata. In particular:

  * it recognises the central importance of URLs for naming things
  * it has built-in support for multiple languages
  * it has good support for distributed extensibility

For this reason it is often treated as a "hub" format, which can be mapped into from RDF/XML and JSON-LD as well as represented in "native RDF" formats such as Turtle. So an obvious option is to define the data model for OpenContracting as an RDF data model and use standard mappings into RDF and back out to other data formats.

Studying the relationship between RDF and XML is informative here. The RDF/XML standard provides a fixed set of rules for mapping a given XML document into an RDF data model. But any given RDF data model can be represented in a number of ways within RDF/XML; XML for the same RDF information can seem very RDF-heavy or fairly close to idiomatic XML depending on the choices made, within a certain scope.

For example, there are some structural rules of RDF/XML that must be adhered to, such as the use of `rdf:about` attributes on elements to indicate that the element is about a particular entity. In a completely idiomatic XML representation, this URL might be indicated by an un-namespaced attribute called `about` or `id`. Similarly, RDF/XML enforces the use of namespaces corresponding to the RDF schemas used within the document, which can mean a lot of namespaces for vocabularies that mix properties from multiple schemas. Idiomatic XML might also add wrapper elements that cannot be used within RDF/XML, or omit wrapper elements that are necessary within an RDF/XML version.

An alternative approach that ensures that information can be represented in idiomatic XML while simultaneously enabling standardised parsing into an RDF data model is to use [GRDDL](TODO). With GRDDL, the XML document, or the namespace document for the vocabulary used within the XML, points to an XSLT stylesheet that describes how the XML should be mapped into an RDF/XML document. Processors that want RDF can use this stylesheet to generate RDF/XML, which they can then parse. This enables the use of completely idiomatic and unconstrained XML that can still be interpreted as RDF.

The goal of having a format that is based on a meta-language such as XML, JSON or CSV is that it is readily understood by the tools that are designed around those formats, and looks familiar to publishers and consumers of those formats. So RDF/XML formats should always be designed to be as close to idiomatic XML as possible, and using GRDDL is usually a better approach. But neither approach tackles the *reverse* transformation, from an RDF data model into an XML representation. This has to be done by custom code.

The upshot is that defining a data model in RDF doesn't provide significant benefits over defining that data model in a narrative form. Very little of a XML &rarr; application &rarr; JSON transformation is aided through the data model being in RDF. The only developers it helps are those whose preferred format is RDF (ie who want to store data in RDF triplestores), and these are probably in the minority for the OpenContracting audience.

#### Data Model

The specification of a data model should be a narrative document accompanied by diagrams that help to describe the model. Concrete syntax should either be avoided, or all the possible syntaxes should be included for any examples in the data model documentation.

> NOTE: Add example pointer to OWL spec?

The data model specification should define a vocabulary: a set of terms and their meaning. However the syntax for these terms should not be fixed in the data model: these are abstract terms which should be mapped into concrete terms within individual syntaxes, while adopting the naming conventions commonly used for the particular syntax. For example, JSON formats commonly use `lowerCamelCase` for the names of properties while XML commonly uses `hyphenated-naming`.

The design of the vocbaulary should include an awareness of existing vocabulary work, in particular:

  * schema.org
  * W3C standard vocabularies such as DCAT, ORG, Data Cube, SKOS
  * IATI

As much as possible, these vocabularies should be adopted. Doing this saves modelling time, increases familiarity for developers, and eases idiomatic mappings to other formats (eg if they can reuse existing vocabularies).

The later discussion on Governance contains suggestions about managing the scope and growth of this data model.

#### Representation in Common Meta-Languages

The two goals for expressing the data model in common meta-languages are:

  1. to express every feature of the data model in the syntax
  2. to do so in a way that feels natural to developers who prefer that syntax

These are best designed by practitioners who are familiar with good vocabulary design in their favoured meta-language. The RDF design is likely to have a 1:1 correspondence with the core data model, while others will have a less tightly bound relationship.

#### Domain-Specific Language

A domain-specific language should be a text-based language designed to articulate contract metadata in a compact and comprehensible way, without the syntactic constraints of particular meta-languages.

### Contracts and Other Documents

Contracts and other documents associated with contracting such as ITTs and proposals are structured, narrative documents, which can run to hundreds of pages. Like other structured documents, there are many formats which could be used for them:

  * Word documents
  * PDFs
  * HTML
  * XML
  * Markdown

For open contracts, open standard formats such as HTML or XML which capture the essential structure of the documents should be favoured, as documents written in these formats are easy to automatically process (eg to perform textual analyses).

There are two main ways of relating contracts with metadata about those contracts:

  * metadata can be embedded within the contract document itself, for example:
    * as document metadata in Word
    * as attachments in PDFs
    * as explicit metadata in the `<head>` of an HTML document
    * as explicit metadata in a defined XML format
    * through RDFa, microdata or microformat markup of HTML or XML documents
    * within a YAML prefix to a Markdown document, as used in Jekyll

  * metadata can be provided separately, with a link from the metadata to the contract document and from the contract document to its metadata (eg through a `<link>` element in HTML, or a `Link` HTTP header)

Embedding metadata helps to prevent mismatches between the information that is available within the document itself and that available as separate metadata. However, it requires the publisher of the metadata to have control over the formatting of the contract document, which may be impossible (eg the author of a contract is frequently different from the publisher of the metadata about that contract).

OpenContracting should define the method of locating metadata about a given document through a method such as:

  1. Gather metadata from as many sources as possible:
     * metadata documents linked to from the `Link` HTTP header
     * metadata documents linked to from within the document (in defined ways for defined formats)
     * metadata embedded within the document itself (extracted in defined ways)
  2. Combine this metadata to get a full picture, with the metadata that is closest to the document (ie embedded within it) having precedence over that defined through other metadata files

## Finding Data

The biggest benefit of having a lot of data using the same syntax is that it can be brought together, enabling comparisons across countries or sectors, and other analyses such as the relationships between organisations.

Organisations that are publishing open data should want that data to be findable. Once the data is located by other organisations, both the data itself and the location of the data should be sharable.

Findability should be considered both at the level of individual contracts (eg if you know that a particular contract exists, you should be able to find it), and at the level of bulk contract information from individual organisations, from governments and internationally.

The section of Governance, below, discusses how to encourage good practices around making OpenContracting data findable. This section focuses on technical approaches.

### Individual Contracts

Individual contracts should be findable using normal web searches, as this is what most users who are looking for individual contracts will use. To ensure contract data is findable through normal web searches, it should be published with normal good SEO practice:

  * available at a persistent URL
  * available in a format that search engines understand, such as HTML
  * including clear descriptive metadata
  * not hidden through mechanisms such as `robots.txt`
  * listed within sitemaps

In addition, if important metadata is marked up appropriately within HTML documents, search engines may provide "rich snippets" &mdash; additional information in search results that help users get high value information quickly. For example, key metadata such as who the contract is between, the start, end dates and duration of the contract, and the price of the contract could be displayed within search results.

The most promising route to get this to happen is to work with schema.org to define a schema.org-based vocabulary for this key contract metadata, around which rich snippets could be designed.

### Bulk Contract Data

Bulk contracting data may be collected at many different levels:

  * by organisations involved in the contracting
  * by local, regional and national governments
  * by national and international non-governmental organisations

#### Patterns for Discovery

There are various patterns that are used for discovering data:

  * A **centralised registry** is a single site at which all data of the required type is registered. An entry in the register includes metadata about the data that enables the registry to be searched for relevant entries. Experience shows that these registries tend to get out of date (eg as data moves to another location on a site) and can become unwieldy for users.
  
  * A **decentralised registry** is a set of registries that are linked together. Entries in one registry may be reflected in other registries, or searches over one registry may result in searches over other registries. Decentralised registries are often conceived of as a separate place in which data needs to be registered, which means that they can have the same drawbacks as centralised registries.
  
  * A **curated list** model is one in which separate organisations or individuals who are interested in the data create and maintain their own lists. This has the benefit that the maintainers of the list are likely to be the users of the list (and therefore have a motivation to keep it up to date) but the disadvantage that the maintainers might not be informed of changes to the data (which means they are less equipped to keep it up to date).
  
  * A **crawler** model is one in which software agents aim to discover data and record its location and characteristics automatically. The crawlers might pick up on information that is deliberately placed to make the data easy to find (such as entries in a sitemap or in a `.well-known` file) or might use more heuristic methods to locate the data. The advantage of this model is that the metadata is automatically kept up to date and in sync with the published data. The disadvantage is that the web is a big place and the crawler might not know where to look: it's likely that for targeted data such as OpenContracting data, there would need to be a maintained list of relevant publishers to focus the crawler's attentions.

#### Recommended Approach

We recommend the following approach:

  1. OpenContracting should define a method for OpenContracting data to be locatable on any domain that publishes OpenContracting metadata. These might be:
     * including pointers within sitemap files
     * including pointers within a file at a `.well-known` location
     * using specific link relations within links on pages on the site
     
  2. Any organisation that publishes OpenContracting data should include pointers to its data using one or more of these methods.
  
  3. Any organisations that wish to create or maintain aggregated views of OpenContracting data should maintain lists of the domains of publishers or pages on their site. These lists might be:
      * maintained manually by trusted members of the organisation
      * maintained in a wiki or other collaboratively maintained source of data
      * created through self-registration of publishing organisations
      
  4. Crawlers should be created that, given one of these lists, crawl and extract relevant data and metadata to build up an aggregated dataset. These crawlers should also flag up missing data so that relevant domains or pages can be removed from the list if they move or disappear.

## Governance

OpenContracting has many stakeholders: governments, NGOs, companies, individuals. The goals and requirements of these different stakeholders are different, which means that the data that they want to see published is different. It is also likely that with time and experience, new and different requirements will come to the fore and become possible to fulfil.

So OpenContracting needs to be a flexible and extensible standard. This sections looks at different models of managing that.

It's noted that previous analysis of published contract information showed very little overlap in the contracting metadata published by different governments.

### Legal Issues

Open standards must be available for anyone to read and to implement, royalty-free. It's important to ensure that any contributions towards the standard (eg text, examples, test files) are unencumbered with IPR. 

### Patterns of Governance

Different patterns are used by different standards organisations:

  * The W3C manages a consensus-based process. Working Groups are chartered to produce standards within a fixed amount of time (usually two years). Standards progress through a recognised process of Working Drafts, Candidate Recommendations, Proposed Recommendations and Recommendations. Objections can be raised (again through a defined process) that prevent the standard from progressing, with ultimate decisions resting with the W3C Director.
  
    Typically a given standard will go through several versions: once the first version of a standard is produced, the Working Group will be re-chartered to produce a second version that adds more features or addresses newly discovered requirements. The old Recommendation remains in place and the new versions reference back to the older versions with lists of major and minor changes.
  
  * The WHAT WG maintains "living documents" for HTML and related standards. These are frequently altered to reflect a combination of group consensus and editorial decisions; the lists of changes are readily available and organisations who wish to implement the standards are expected to be actively involved through mailing lists in order to keep up to date with the current thinking about the design of the standard. The only versioning that is done is through references to particular commits within a version control system.
  
  * microformats are developed and maintained through a defined process [TODO]
  
  * schema.org is maintained by a group of organisations and while the vocabularies are open to anyone to use, the process that determines their content is more opaque.

### Recommended Approach

The most important thing about a standard is its implementation. Implementations of a standard format may be tools that help with production of the data such as

  * editors
  * validators
  * publishing platforms

or tools that help with the interpretation of the data such as

  * visualisers
  * analytical algorithms

Of these, the applications that aid the interpretation of the data are more important: they determine how the data is used and thus what data is useful.

It is usual for different implementations, and particularly different visualisations or analysis, to use different metadata. For example, a visualisation of the distribution of contracts to different sizes of companies within a particular country will rely on metadata about the worth of contracts and the companies that they are awarded to. An international comparison of the amount of work being done in different sectors will need to know the size and the sector of each contract for a set of countries.

We recommend a method through which:

  1. OpenContracting identifies and defines communities of practice which have different analytical goals or purposes to which the data will be put. There should be a small number of these (around five) to begin with.
  2. Each community of practice identifies the metadata that is required to enable their particular purpose.
  3. OpenContracting identifies a minimal core set of metadata that is required by all communities of practice; it's likely that this is a very limited set, comprising just an identifier for the contract and a title.
  4. Publishers of OpenContracting data should declare (in a machine-readable way, but also through iconography on their contracting websites) which purposes they aim to satisfy through the data that they publish.
  5. Developers of tools that work with OpenContracting data should declare (in a machine-readable way but also through iconography on their websites) their purpose(s), ie which sets of metadata are required for the tool to work. This makes it clear to users of the tools which data can be analysed with it, and to publishers which metadata needs to be published so that they can use the tool.

To facilitate this, OpenContracting should:

  * Maintain a data model / vocabulary that encompasses the superset of all the metadata required by all communities of practice. This enables new communities of practice to standardise on existing terms & definitions rather than inventing their own.
  
  * Maintain a registry of "purposes" and the metadata that is required to satisfy each of those purposes, so that publishers know what they need to publish.
  
  * Support a process by which new communities of practice can form, identify a purpose, and register it along with a set of required metadata for that purpose which is incorporated into the OpenContracting data model. It's recommended that each purpose can have a status (eg proposed vs implemented) and that purposes are only marked as implemented when there are implementations that use the given subset of data for the specified purpose. This ensures that the set of required metadata is the right one.
  
  * Review the data model, purposes and published data on a regular basis to see whether the core standard vocabulary for OpenContracting should be expanded.