
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
