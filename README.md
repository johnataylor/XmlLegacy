# Addressing the XML Legacy

## We want JSON but we have XML.

Imagine we start with the following XML that describes one of the books we have in our library.

    <?xml version="1.0" encoding="utf-8" ?>
      <book xmlns="http://schemas.example.org/library">
        <title>Semantic Web for the working ontologist: effective modeling in RDFS and OWL</title>
        <authors>
          <author>Allemang, Dean</author>
          <author>Hendler, Jim</author>
        </authors>
        <isbn>978-0-12-385965-5</isbn>
        <publisher>Morgan Kaufmann</publisher>
        <published>2011</published>
    </book>

It's great that we have this data in a structured format, however, sadly, it's the wrong format, because we need JSON. JSON is a natural choice because, firstly, we need to make it available to our Web programmers who would like to consume it in JavaScript running in the Web browser. And secondly we will undoubtedly have another group within our organization that wants to apply some big-data style analytics. And this group will have recently adopted JSON as their favored data format because they have found that it works best with the various server-side scripting technologies they make heavy use of.

To create the JSON we are going to take a slightly roundabout route, the benefits of which will hopefully become clear as we develop more recipes. Specifically we are first going to translate this XML into a flattened graph serialization model called RDF. We don't need to be too concerned about all the details of RDF right now. Sufficient to say that RDF is a triple set data representation where the nodes and arcs of our graph are identified with URIs. RDF is not a data format exactly, rather it is a conceptual model that has in turn multiple well defined data formats. One of these, from many years ago, is XML based and another of which, from much more recently, is JSON based. The interesing thing about the JSON serialization is that it can be made to look like normal, regular JSON where conceptual objects in our model are actual objects in the JSON and conceptual properties in our model are actual properties in the JSON. This might sound all rather obvious, but there is a surprising amount of JSON kicking around that doesn't follow these basic rules and therefore, doesn't bind to programming languages like JavaScript in the seamless way you might have hoped for. And losing the seamless binding to code rather loses much of the point of adopting JSON in the first place.

For reference the XML representation is called RDFXML and the JSON representation is called JSON-LD. Both of these are industry standards. But to get started we are going to dig up another old industry standard, something of a blast-from-past perhaps, good old XSLT.

The first step is to write an XSLT transform that takes the XML snippet and turns it into an RDFXML document. It turns out that this is a fairly straightforward exercise, that is once we have got our heads around the quirks of RDFXML and reminded ourselves how to write XSLT. Here is our solution:

    <?xml version="1.0" encoding="utf-8" ?>
    <xsl:stylesheet
      xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
      xmlns:src="http://schemas.example.org/library"
      xmlns:library="http://schemas.example.org/library#"
      xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
      version="1.0">
      <xsl:param name="baseAddress" />
      <xsl:template match="/src:book">
        <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
          <library:Book>
            <xsl:attribute name="rdf:about">
              <xsl:value-of select="concat($baseAddress, src:isbn, '.json')"/>
            </xsl:attribute>
            <xsl:for-each select="*">
              <xsl:choose>
                <xsl:when test="self::src:authors">
                  <xsl:apply-templates select="src:author" />
                </xsl:when>
                <xsl:otherwise>
                  <xsl:element name="{concat('library:', local-name())}">
                    <xsl:value-of select="."/>
                  </xsl:element>
                </xsl:otherwise>
              </xsl:choose>
            </xsl:for-each>
          </library:Book>
        </rdf:RDF>
      </xsl:template>
      <xsl:template match="src:author">
        <library:author>
          <library:Author>
            <xsl:attribute name="rdf:about">
              <xsl:value-of select="concat($baseAddress, 'author/', ., '.json')" />
            </xsl:attribute>
            <library:name>
              <xsl:value-of select="."/>
            </library:name>
          </library:Author>
        </library:author>
      </xsl:template>
     </xsl:stylesheet>

Looking past the slightly weird syntax of XSLT we should note that there is something really significant going on here. What we are actually doing is manufacturing unique identifiers, specifically URIs, that represent the key domain concepts captured in that XML. Specifically, our little transformation creates URIs for the book and each of the authors. We were lucky enough here to be able to use properties we found in the data, but whatever tricks we find ourselves having to do, we should remember that it is the assignment of URIs to concepts buried in our XML that is the key to much of what follows. We don't actual care to much about the particular shape of the URIs, but what we do care about is that we have a consistent and repeatable way of creating them.

Anyhow, we said we were going to create JSON. Well as it turns out, we are just about there already. That is, now we have an RDFXML document we can load it into a RDF graph object and then just save it right out again, only this time, telling the system to save it as JSON. The only addition step we might need is to give that final process some hints so we can precisely control the final space the JSON takes. We do this by defining a JSON-LD context. This context can actually be of benefit to various agents that might like to process this JSON so its a good idea to keep it attached to the JSON we will produce, there are various ways to do this, but perhaps the simplest place to start is to just include it in the JSON under a special property that goes by the name of "@context". Here is the resulting JSON:

    {
      "@id": "http://example.org/book/978-0-12-385965-5.json",
      "@type": "Book",
      "authors": [
        {
          "@id": "http://example.org/book/author/Allemang,%20Dean.json",
          "@type": "Author",
          "name": "Allemang, Dean"
        },
        {
          "@id": "http://example.org/book/author/Hendler,%20Jim.json",
          "@type": "Author",
          "name": "Hendler, Jim"
        }
      ],
      "isbn": "978-0-12-385965-5",
      "published": "2011",
      "publisher": "Morgan Kaufmann",
      "title": "Semantic Web for the working ontologist: effective modeling in RDFS and OWL",
      "@context": {
        "@vocab": "http://schemas.example.org/library#",
        "authors": {
          "@id": "author",
          "@container": "@set"
        },
        "books": "@graph"
      }
    }

Generally code that deals with JSON is very good at ignoring what it doesn't understand and so we shouldn't be concerned about the various "@" properties; they are mainly there just to help disambiguate. Interestingly because they are there there was no accidental information loss from the original XML: we could easily reverse the process and create the XML we started with if we so desired.  

As is often the case when we decide to align our work with industry standards we find that there are many high quality open source implementations available on most widely used platforms. As we are working in .NET we will be using the DotNetRdf library which we can easily download from NuGet.

## We want a single way to represent data in our organization.

However even when we have data in XML we still have multiple different way for that data to be represented in the XML. Perhaps we should stop obsessing over the format and start focusing more on the concepts. Here is the same book, but expressed in a very different peice of XML.

    <?xml version="1.0" encoding="utf-8" ?>
    <book xmlns="http://schemas.example.org/alt/library" 
          isbn="978-0-12-385965-5"
          title="Semantic Web for the working ontologist: effective modeling in RDFS and OWL" >
      <author name="Allemang, Dean" />
      <author name="Hendler, Jim" />
      <published by="Morgan Kaufmann" in="2011" />
    </book>

The developer who came up with this XML clearly favored the more compact representation that can be had with using attributes but the net effect is really only that this XML snippet is just different, equally valid perhaps, but different all the same. And of course, we would really like to understand that these two peices of XML are actually talking about the same thing. So what we need to do then is normalize. And rather than picking a winner between these two formats instead we are going to focus on the concepts, the real semantics, in the data - and that means translating this XML into RDF. Here is the transformation:

    <?xml version="1.0" encoding="utf-8" ?>
    <xsl:stylesheet
      xmlns:xsl="http://www.w3.org/1999/XSL/Transform"
      xmlns:src="http://schemas.example.org/alt/library"
      xmlns:library="http://schemas.example.org/library#"
      xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
      version="1.0">
      <xsl:param name="baseAddress" />
      <xsl:template match="/src:book">
        <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#">
          <library:Book>
            <xsl:attribute name="rdf:about">
              <xsl:value-of select="concat($baseAddress, @isbn, '.json')" />
            </xsl:attribute>
            <xsl:if test="@title">
              <library:title>
                <xsl:value-of select="@title" />
              </library:title>
            </xsl:if>
            <xsl:for-each select="*">
              <xsl:choose>
                <xsl:when test="self::src:author">
                  <library:author>
                    <library:Author>
                      <xsl:attribute name="rdf:about">
                        <xsl:value-of select="concat($baseAddress, 'author/', @name, '.json')" />
                      </xsl:attribute>
                      <library:name>
                        <xsl:value-of select="@name"/>
                      </library:name>
                    </library:Author>
                  </library:author>
                </xsl:when>
                <xsl:when test="self::src:published">
                  <xsl:if test="@by">
                    <library:publisher>
                      <xsl:value-of select="@by" />
                    </library:publisher>
                  </xsl:if>
                  <xsl:if test="@in">
                    <library:published>
                      <xsl:value-of select="@in" />
                    </library:published>
                  </xsl:if>
                </xsl:when>
              </xsl:choose>
            </xsl:for-each>
          </library:Book>
        </rdf:RDF>
      </xsl:template>
    </xsl:stylesheet>

Again, like the previous XSLT, the main thing going on here is the assignment of URIs to the key concepts in our domain; in our example that means books and authors. Although this XSLT is different than the previous example in our first recipe the way it manufactures the URIs is exactly the same. In fact this XSLT produces exactly the same graph from the XML at the start of this recipe as the previous XSLT did for its XML. And because this XSLT produces the same graph it will produce the same JSON.  

The attentive reader might feel a little cheated realizing that the recipe presented here is just the previous one but with different ingredience. This is certainly true, however, within that fact there is a realization that what is going on here is a process of normalization. Instead of simply translating one shape of XML into another, we instead focused on identifyign and labelling the key concepts in the documents. And having labeled the key concepts the rest of the model is simplicity itself. All we do is associate properties with those identified concepts, the properties have values and that's our raw data, and naturally, sometimes, the property value can be another identified concept, and that's our structure covered.


## Sometimes it difficult to find the right words.

Picking the right words matters, but only so much. Its really more important, at least at first, to just be consistent. Looking back at our two previous examples the other thing happening here is that we made an effort to use the same vocabulary. When we say "vocabulary" we simply mean the names of properties we chose to use.

Realizing that the ultimate value in our data comes when we network it and combine it with other data its also natural that we use namespaces for our property names, that will certainly avoid any future conflicts. Namespaces turn out to be one of the most powerful simplifying techniques in software, raw JSON lacks explicit namespaces, however, raw JSON has also typically been served up from a REST endpoint and as such effortlessly inherited an implicit namespace. JSON-LD strengthens this significantly and introduces namespaces explicitly in the document, but manages to do so in a light weight non-invasive way that shouldn't cause any JavaScript programmers to trip up.

## To see the full picture sometimes we need to combine information about a topic from different sources.


## Rearranging and pivoting the data can help to simplify the consuming code.


## Discovering more relationships within the data can help to significantly simplify our business logic.




