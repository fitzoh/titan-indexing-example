this weird experiment of (semi-) literate programming is meant to illustrate the mechanism titan uses to map it's property keys to elasticsearch and it's other indexing options, and answer questions such as this one: https://groups.google.com/forum/#!topic/aureliusgraphs/m4bZM51-Gwg


to run: 
    ensure nothing is running on port 9200
    gradle run


example output (with logging removed):
deleting any existing data
configuring and opening a graph on berkeley and embedded es
	g = standardtitangraph[berkeleyje:/tmp/graph]

next we open the management to create a property and index it
	mgmt = com.thinkaurelius.titan.graphdb.database.management.ManagementSystem@9accff0
	propKey = prop
	idx = byPropKey

then we create a vertex with the property key
	v = v[256]
	p = vp[prop->value]
and give ES a second to index it


verify that the graph has a single vertex
	count = 1
and that it has the expected properties
	props = [prop:[vp[prop->value]]]

now we query ES to see what it has indexed
	results = [_shards:[failed:0, successful:5, total:5], hits:[hits:[[_id:74, _index:titan, _score:1, _source:[1l1:value], _type:byPropKey]], max_score:1, total:1], timed_out:false, took:1]
we can inspect the doc it found
	doc = [_id:74, _index:titan, _score:1, _source:[1l1:value], _type:byPropKey]

you'll notice that the id of the document and the id of the vertex don't match
	docId == v.id = false
you'll also notice that the property is mapped with an unexpected key
	docSrc = [1l1:value]

however, if you run the source doc id through the LongEncoder decoder:
	decodedDocId = 256
you'll see that it now matches the vertex id
	{decodedDocId == v.id()} = true

likewise, if we take the key from the source document
	src = 1l1
and decode it
	decodedKey = 2053
we'll end up with an integer that doesn't seem to have anything to do with the string 'prop'
it does however match the internal id of the property key we created earlier
	propKeyId = 2053
	{propKey.id() == decodedKey} = true

to recap, the vertex id is run through a long encoder, and stored in es as the document id
and the id of the property key object is encoded and stored as the key for the property in the document

if we want to query elasticsearch directly for a property key value
	{LongEncoding.encode(PropKey.id()} = 1l1
	queryString = 1l1:value
	params = [q:1l1:value]

make the query
and you'll notice that this gives the same result as the first query, '*:*'
	results2 = [_shards:[failed:0, successful:5, total:5], hits:[hits:[[_id:74, _index:titan, _score:0.30685282, _source:[1l1:value], _type:byPropKey]], max_score:0.30685282, total:1], timed_out:false, took:63]


