
## Sometimes it difficult to find the right words.

Picking the right words matters, but only so much. Its really more important, at least at first, to just be consistent. Looking back at our two previous examples the other thing happening here is that we made an effort to use the same vocabulary. When we say "vocabulary" we simply mean the names of properties we chose to use.

Realizing that the ultimate value in our data comes when we network it and combine it with other data its also natural that we use namespaces for our property names, that will certainly avoid any future conflicts. Namespaces turn out to be one of the most powerful simplifying techniques in software, raw JSON lacks explicit namespaces, however, raw JSON has also typically been served up from a REST endpoint and as such effortlessly inherited an implicit namespace. JSON-LD strengthens this significantly and introduces namespaces explicitly in the document, but manages to do so in a light weight non-invasive way that shouldn't cause any JavaScript programmers to trip up.

Something that can really help when we start to share our data is adopting a common vocabulary. Luckily there are some industry standard vocabularies we are free to just pick up and use. There really is no downside to using a standard vocabulary. A great example is the Dublin Core vocabulary that includes a bunch of terms used to describe published artifacts. (Dublin Core gained a little traction when it came out, for example, not many people realize that Microsoft Office documents actually bury Dublin Core metadata in their document format.)

One of the key motivations for adopting a standard vocabulary is that it allows our data to be consumed by agents we had not anticipated. For example, search engines, whether public or private are immediately able to be just that little bit smarter about indexing our dcouments. That can't be a bad thing.

Perhaps a little ironically the example we introduced in our first couple of recipes was metadata to describe a published book. In hindsight we could have just used Dublin Core rather than struggling to find our own words. The good news is the mistake is easily corrected. All we need to do if we decide now to adopt after the fact to adopt the vocabulary we missed out on we can just add it in by evaluating a series of rules that express the equivalance. 


