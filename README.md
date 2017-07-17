# Overview

The system described here is designed to process linked data search results into a format that is easily digestable by end applications.  It allows user interfaces to be designed to handle searching and presentation of linked data in a consistent way.  

There are a number of authorities that are configured to work with this process.  Look for all repositories starting with `qa_` in [Linked Data for Libraries Labs (LD4L Labs)](https://github.com/ld4l-labs).

There are two servers that drive this system.

1. linked data authority server - This server can be controlled by the authority provider or you can use a locally cached version of the authority.  It must be querable via curl with the query passed as a parameter.  Additionally, it must return a serialization of rdf (e.g. rdf/xml, jsonld, ntripled, turtle, etc.)

2. questioing authority normalization server - This server is an installing of the [questioning authority (qa) gem](https://rubygems.org/gems/qa).  A configuration file specific for NALT is installed which tells QA how to extract information from the linked data search results in order to normalize the results that are passed to the application.

More information on working with linked data through the QA gem is available in it's README in section [Linked Open Data (LOD) Authorities](https://github.com/samvera/questioning_authority#linked-open-data-lod-authorities)


## Setting up a linked data authority server for NALT

### Use the external NALT authority server

At the time of this writing, NALT does not provide a linked data search API.

### Use a local cache of NALT authority data

You can create a local cache of NALT authority data by [downloading]() it into a local triplestore.  There are a number of triplestores that you can use for this purpose.

You may want to consider using [SKOSMOS](http://skosmos.org/) to hold NALT data.  It is optimized for [SKOS ontology](https://www.w3.org/TR/2008/WD-skos-reference-20080829/skos.html) based linked data.


## Setting up a QA server to include the NALT authority

### Create a QA server app IF this is your first authority

Create a rails app and install the QA gem by following the instructions in the QA gem in section [How do I use this?](https://github.com/samvera/questioning_authority#how-do-i-use-this)

NOTE: Hyrax applications have QA built in.  You can follow the instructions for an existing QA server with the Hyrax app acting as the server.  You still might want to create a separate QA server app that lives on an optimized machine or to share the authority access between multiple apps.  See [Hyrax documentation](https://github.com/samvera/hyrax/wiki/Autocomplete-in-Hyrax) for more information.

### For existing and new QA servers

1. Add the configuration for NALT from [config/authorities/linked_data/nalt.json](https://github.com/ld4l-labs/qa_nalt/tree/master/config/authorities/linked_data/nalt.json) to the same directory location in your server app.
1. Edit nalt.json 
  1. update `search -> url -> template` and `term -> url -> template` to be the URL to the linked data authority server that you are using.  
  1. If you are using a SKOSMOS based server, then you shouldn't have to change any other configuration settings.  If using a different triplestore, then the parameters for limiting the number of results and filtering languages may vary and require an update to the `search -> url -> mapping` for the search URL parameters and `term -> url -> mapping` for the term URL parameters. 

More information on writing QA linked data configurations is available in it's README in section [Configuring a LOD Authority](https://github.com/samvera/questioning_authority#configuring-a-lod-authority).

## Debugging

Because there are a number of systems integrating with each other, you should test the generated links at each level.

### Test that the linked data authority server returns expected RDF serialization.

For a SKOSMOS server, this will be something like...

http://localhost:3005/rest/v1/nalt/search?query=*rice*&lang=en&maxhits=3

```
{"@context":
  {"skos":"http:\/\/www.w3.org\/2004\/02\/skos\/core#",
   "isothes":"http:\/\/purl.org\/iso25964\/skos-thes#",
   "onki":"http:\/\/schema.onki.fi\/onki#",
   "uri":"@id",
   "type":"@type",
   "results":
     {"@id":"onki:results",
      "@container":"@list"
     },
   "prefLabel":"skos:prefLabel",
   "altLabel":"skos:altLabel",
   "hiddenLabel":"skos:hiddenLabel",
   "@language":"en"
  },
 "uri":"",
 "results":
   [{"uri":"http:\/\/lod.nal.usda.gov\/nalt\/1293",
     "type":["skos:Concept"],
     "localname":"nalt\/1293","prefLabel":"16SrXI (Rice yellow dwarf group)",
     "lang":"en",
     "vocab":"nalt"
    },
    {"uri":"http:\/\/lod.nal.usda.gov\/nalt\/1892",
     "type":["skos:Concept"],
     "localname":"nalt\/1892",
     "prefLabel":"Acacia holosericea",
     "lang":"en",
     "vocab":"nalt"
    },
    {"uri":"http:\/\/lod.nal.usda.gov\/nalt\/1892",
    "type":["skos:Concept"],
    "localname":"nalt\/1892",
    "prefLabel":"Acacia holosericea",
    "lang":"en",
    "altLabel":"Acacia holosericea A. Cunn.",
    "vocab":"nalt"
   }
  ]
}    
```

### Test that QA returns the expected normalized data

For the attached configuration, with the server running on port 3002 on your local machine, this will be something like...

http://localhost:3002/qa/search/linked_data/nalt?q=test&maxRecords=3

```
[
  {"uri":"http://lod.nal.usda.gov/nalt/5133",
   "id":"http://lod.nal.usda.gov/nalt/5133",
   "label":"agglutination tests"
  },
  {"uri":"http://lod.nal.usda.gov/nalt/7182",
   "id":"http://lod.nal.usda.gov/nalt/7182",
   "label":"animal use alternatives"
  },
  {"uri":"http://lod.nal.usda.gov/nalt/1841",
   "id":"http://lod.nal.usda.gov/nalt/1841",
   "label":"intestinal absorption"}]
```

#### Test that QA generates the expected linked data authority server URL

If you don't see any results, you can check that the correct URL for the linked data server is generated.  In the QA server, search log/development.log for `QA Linked Data search url:`.  Then you can copy/paste that URL into a browser or use curl in the terminal to verify that the generate URL accessing the linked data authority server actually returns data as expected.
