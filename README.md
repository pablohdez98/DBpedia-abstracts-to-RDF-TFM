# DBpedia-abstracts-to-RDF
Web application to convert DBpedia abstracts into a set of RDF triples.

This is a Gsoc project in colaboration with DBpedia, more information can be found [here][1].

During the development of the GSoC project, a blog has been created with weekly entries explaining how the tool has been built and what difficulties we have encountered along the way. **This blog can be accessed [here][2].**

In addition, there is a folder [codingweeks][3] where there is a python script for each week with the pipeline developed so far.

## Pipeline overview 
The tool performs the following steps to translate from text to RDF:
1.  Clean up the text superficially (remove some parenthesis and special characters).
2.  Apply the dependency parse tree to the original text.
3.  Split the text into sentences
4.  Identify which sentences are simple and which are complex (more than one verb in them).
5.  For each simple sentence create text triples that follow the structure `subject | predicate | object` searching from the root of the parse tree for possible subjects and objects.
6.  For each complex sentence try to simplify into several simple sentences and execute step 5 with these simple sentences.
7.  Process the text triples to fit the desired structure.
8.  Apply DBpedia Spotlight on the original text to create a dictionary with those resources identified by NER.
9.  For each text triplet replace the subject and object with the resources (URIs) identified by Spotlight (if they have been identified).
10. To translate the predicate, a lookup table is used to search by verb + preposition and obtain a property from the DBpedia ontology.
11. If the verb of the predicate is the verb `to be` the property will be `rdf:type` and the object will be a DBpedia class. For this another lexicalization table is needed.
12. Finally the RDF triples with the structure `resource | property | resource` are obtained if there has not been any lexicalization problem. In the case in which a valid lexicalization is not found we will find a `literal` in the triple.

This is a brief summary, if you want more information visit the [blog][2] where the whole development is described.

## License
Copyright (C) Fernando Casabñan Blasco, 2021

This program is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with this program. If not, see http://www.gnu.org/licenses/.

## Requirements
The first step is to install the requirements:
```console
python -m pip install -r requirements.txt
```

In principle the spacy model used for the dependency tree is included in the list of requirements, if there is any problem with the installation of the model run the following command:
```console
python -m spacy download en_core_web_trf
python -m spacy download en_core_web_lg
```

The model ending in `_lg` needs to be installed as it is a requirement to be able to use the coreferee library to deal with coreferences.
As with the spacy models, if there is a problem with the installation of coreferee when using the requirements list run the following commands:
```console
python3 -m pip install coreferee
python3 -m coreferee install en
```

The next step, which is optional, would be to install the local DBpedia Spotlight server. To do so, just download the model and the server file from the following [tutorial][7]. It is also explained in detail in the [spotlight][8] folder.

If you do not want to install the tool you can make use of the Spotlight online API, being this slower and with limited queries. The pipeline is designed so that if you cannot connect to the local server you can use the online API, however you can specify directly to use the online API by modifying an argument of the `brt.get_annotated_text_dict(raw_text, service_url=SPOTLIGHT_LOCAL_URL)` function that can be found in the pipeline.
You should replace `service_url=SPOTLIGHT_LOCAL_URL` with `service_url=SPOTLIGHT_ONLINE_API`.

## Usage
The application has two interfaces: the web interface and the command line interface.

To run the web application just use the following command:
```console
streamlit run code/webapp/app.py
```

On the other hand, if you want to use the command line application use the following command:
```console
python3 code/app/app.py --input_text="code/app/input.txt" --all_sentences --save_debug
```

Both alternatives require three configuration files at your fingertips. These are the verb lexicalization table [`verb_prep_property_lookup.json`][4], the DBpedia class lexicalization table [`classes_lookup.json`][5] and the DBpedia ontology [`dbo_ontology_2021.08.06.owl`][10]. If there are any problems reading any of these files please modify the paths to these files as you see fit.

For more information visit [blog post 10][9] where the different arguments that exist when executing any interface are explained.

## Future work (what is left to do)
The main problem of the pipeline right now is that of all the triples it can generate, only half of them are free of errors in their structure. This kind of errors occur when the pipeline is unable to find a valid lexicalization for the predicate (verb+preposition) or for the object, leaving these elements as Literals instead of as resources or properties.
When we mention lexicalization we refer to the translation of text to URIs using tables for [properties][4] and [DBpedia classes][5].
As already seen during [blog post 9][6] the origin of most of these errors is when lexicalizing verbs and prepositions, since the current state of the lookup table does not cover the estimated number of cases.

Because of the above, the next step in the development of this tool would be to collaboratively add many more translations, so we will be able to cover many more cases and therefore generate more RDF triples.

Once many more lexicalizations are added we will be able to be stricter when generating RDF triples because at the moment there are also some errors associated with the content of the triple. This occurs when the range or domain of the RDF property does not match the classes of the resources in the subject and predicate. The reason for this error is again the fact that there are hardly any property lexicalizations at the moment.

Also the text processing part should be reworked, especially when dealing with more complex sentences. Complex sentences are those that have more than one verb and must be simplified before translating them into RDF.

Another future line of development to consider would be to support more languages such as Spanish, German, etc.

Finally, it would also be nice to implement a system to always have the lexicalization tables updated, so that each time the application is run it checks if these tables are in the latest possible version.

[1]: https://summerofcode.withgoogle.com/projects/#6560166180290560
[2]: https://fcabla.github.io/DBpedia-abstracts-to-RDF/
[3]: https://github.com/Fcabla/DBpedia-abstracts-to-RDF/tree/main/code/codingweeks
[4]: https://github.com/Fcabla/DBpedia-abstracts-to-RDF/blob/main/datasets/verb_prep_property_lookup.json
[5]: https://github.com/Fcabla/DBpedia-abstracts-to-RDF/blob/main/datasets/classes_lookup.json
[6]: https://fcabla.github.io/DBpedia-abstracts-to-RDF/coding-week9
[7]: https://github.com/dbpedia-spotlight/dbpedia-spotlight/wiki/Run-from-a-JAR
[8]: https://github.com/Fcabla/DBpedia-abstracts-to-RDF/tree/main/spotlight
[9]: https://fcabla.github.io/DBpedia-abstracts-to-RDF/coding-week10
[10]: https://github.com/Fcabla/DBpedia-abstracts-to-RDF/blob/main/datasets/dbo_ontology_2021.08.06.owl
