# Addressing the XML Legacy



## Sometimes it difficult to find the right words.

Picking the right words matters, but only so much. Its really more important, at least at first, to just be consistent. Looking back at our two previous examples the other thing happening here is that we made an effort to use the same vocabulary. When we say "vocabulary" we simply mean the names of properties we chose to use.

Realizing that the ultimate value in our data comes when we network it and combine it with other data its also natural that we use namespaces for our property names, that will certainly avoid any future conflicts. Namespaces turn out to be one of the most powerful simplifying techniques in software, raw JSON lacks explicit namespaces, however, raw JSON has also typically been served up from a REST endpoint and as such effortlessly inherited an implicit namespace. JSON-LD strengthens this significantly and introduces namespaces explicitly in the document, but manages to do so in a light weight non-invasive way that shouldn't cause any JavaScript programmers to trip up.

## To see the full picture sometimes we need to combine information about a topic from different sources.


## Rearranging and pivoting the data can help to simplify the consuming code.


## Discovering more relationships within the data can help to significantly simplify our business logic.




