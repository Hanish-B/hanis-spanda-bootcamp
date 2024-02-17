<!-----

You have some errors, warnings, or alerts. If you are using reckless mode, turn it off to see inline alerts.
* ERRORs: 0
* WARNINGs: 0
* ALERTS: 4

Conversion time: 1.383 seconds.


Using this Markdown file:

1. Paste this output into your source file.
2. See the notes and action items below regarding this conversion run.
3. Check the rendered output (headings, lists, code blocks, tables) for proper
   formatting and use a linkchecker before you publish this page.

Conversion notes:

* Docs to Markdown version 1.0β35
* Wed Jan 31 2024 21:24:22 GMT-0800 (PST)
* Source doc: spanda-NaLLM-5
* This document has images: check for >>>>>  gd2md-html alert:  inline image link in generated source and store images to your server. NOTE: Images in exported zip file from Google Docs may not appear in  the same order as they do in your doc. Please check the images!


WARNING:
You have 3 H1 headings. You may want to use the "H1 -> H2" option to demote all headings by one level.

----->


<p style="color: red; font-weight: bold">>>>>>  gd2md-html alert:  ERRORs: 0; WARNINGs: 1; ALERTS: 4.</p>
<ul style="color: red; font-weight: bold"><li>See top comment block for details on ERRORs and WARNINGs. <li>In the converted Markdown or HTML, search for inline alerts that start with >>>>>  gd2md-html alert:  for specific instances that need correction.</ul>

<p style="color: red; font-weight: bold">Links to alert messages:</p><a href="#gdcalert1">alert1</a>
<a href="#gdcalert2">alert2</a>
<a href="#gdcalert3">alert3</a>
<a href="#gdcalert4">alert4</a>

<p style="color: red; font-weight: bold">>>>>> PLEASE check and correct alert issues and delete this message and the inline alerts.<hr></p>


_This is the fifth blog post of Neo4j’s NaLLM project. We started this project to explore, develop, and showcase practical uses of these LLMs in conjunction with Neo4j. As part of this project, we have constructed and publicly displayed demonstrations in a [GitHub repository](https://github.com/neo4j/NaLLM), providing an open space for our community to observe, learn, and contribute. Additionally, we have been writing about our findings in a blog post. You can read the previous four blog posts here:_



* _[Harnessing LLMs with Neo4j](https://medium.com/neo4j/harnessing-large-language-models-with-neo4j-306ccbdd2867)_
* _[Fine-tuning vs. Retrieval-augmented generation](https://medium.com/neo4j/knowledge-graphs-llms-fine-tuning-vs-retrieval-augmented-generation-30e875d63a35)_
* _[Multi-hop Question Answering](https://medium.com/neo4j/knowledge-graphs-llms-multi-hop-question-answering-322113f53f51)_
* _[Knowledge Graphs & LLMs: Real-Time Graph Analytics](https://medium.com/neo4j/knowledge-graphs-llms-real-time-graph-analytics-89b392eaaa95)_

This blog post will explore a use case we investigated during our project: extracting information from unstructured data. Organizations have long faced challenges in extracting meaningful insights from unstructured data. Such data encompasses textual content, images, audio, and other non-tabular formats, holding immense potential yet often remaining difficult to use due to its inherent complexity. Our primary focus in this post will be to extract information from unstructured text by converting it into nodes and relationships.



<p id="gdcalert1" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image1.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert2">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image1.png "image_tooltip")


Illustration from [Imagine.art](http://imagine.art/) of “extracting information from unstructured data”

Recent years have witnessed significant advancements in natural language processing techniques, revolutionizing the transformation of unstructured data into valuable knowledge. With the emergence of powerful language models like OpenAI’s GPT models and leveraging the power of machine learning, the process of converting unstructured text data into structured representations has become more accessible and efficient.

One such representation is **knowledge graphs,** which offer a robust framework for representing complex relationships and connections among various entities. They provide a structured representation of the data, enabling intuitive querying and exploration of the information contained within. This structured nature allows for advanced semantic analysis, reasoning, and inference, facilitating more accurate and comprehensive decision-making processes.



<p id="gdcalert2" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image2.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert3">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image2.png "image_tooltip")


Example of Knowledge Extraction Pipeline

We will explore how Large Language Models (**LLMs**) have simplified the conversion of unstructured data into knowledge graphs, using an approach that utilizes the language skills of LLMs to perform nearly all parts of the process. The process can be divided into three steps:



1. Extracting nodes and edges
2. Entity disambiguation
3. Importing into Neo4j

Let’s walk through each of these steps:

**1. Extracting nodes and relationships:** To tackle this problem, we take the simplest possible approach, where we pass the input data to the LLM and let it decide which nodes and relationships to extract. We ask the LLM to return the extracted entities in a specific format, including a name, a type, and properties. This allows us to extract nodes and edges from the input text.

However, LLMs have a limitation known as the **_context window_** (between 4 and 16,000 tokens for most LLMs), which can be easily overwhelmed by larger inputs, hindering the processing of such data. To overcome this limitation, we employ a strategy of dividing the input text into smaller, more manageable chunks that fit within the context window.

Determining the optimal splitting points for the text is a challenge of its own. To keep things simple, we have chosen to divide the text into chunks of maximum size, maximizing the utilization of the context window per chunk. Additionally, we introduce some overlap from the previous chunk to account for cases where a sentence or description spans across multiple chunks. This approach allows us to extract nodes and edges from each chunk, representing the information contained within it.

To maintain consistency in the labeling of different types of entities across chunks, we provide the LLM with a list of node types that were extracted in the previous chunks. Those start forming the extracted “schema.” We have observed that this approach enhances the uniformity of the final labels. For example, instead of the LLM generating separate types for “Company” and “Gaming Company,” it consolidates all types of companies under a “Company” label.

One **_notable hurdle_** in our approach is the problem of duplicate entities. Since each chunk is processed semi-independently, information about the same entity found in different chunks will create duplicates when we combine the results. Naturally, this issue brings us to our next step.

**2. Entity disambiguation:** We now have a set of entities. To address the issue of duplication, we employ LLMs once again. First, we organize the entities into sets based on their type. Subsequently, we provide each set to the LLM, enabling it to merge duplicate entities while simultaneously consolidating their properties. We use LLMs for this since we don’t know what name each entity has been given. For example, the initial extraction could have ended up with two nodes: (Alice {name: “Alice Henderson”}) and (Alice Henderson {age: 25}). These reference the same entity and should be merged to a single node with both the name and age property. We use LLMs to accomplish this since it’s great at quickly understanding which nodes actually reference the same entity.

By iteratively performing this procedure for all entity groups, we obtain a structured data set that is ready for further processing.

**3. Importing the data into Neo4j:** In the final step of the process, we focus on importing the results we got from the LLM into a Neo4j database. This requires a format that Neo4j can understand. To accomplish this, we parse the generated text from the LLM and transform it into separate CSV files, corresponding to the various node and relationship types. These CSV files are subsequently mapped to a format compatible with the [Neo4j Data Importer tool](https://data-importer.neo4j.io/connection/connect). Through this conversion, we gain the advantage of previewing the data before initiating the import process into a Neo4j database, harnessing the capabilities offered by the Neo4j Importer tool.



<p id="gdcalert3" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image3.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert4">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image3.png "image_tooltip")


Overview of the application

Putting this all together, we have created an application consisting of three parts: a UI to input a file, a controller that executes the previously explained process, and an LLM that the controller talks to. This demo application can be found [here](https://nallm-experiments.ew.r.appspot.com/), and the source code can be found on [GitHub](https://github.com/neo4j/NaLLM).

We also created a version of this pipeline that works essentially in the same way but with the option to include a schema. This schema works like a filter where the user can restrict which types of nodes and relationships and which properties the LLM should include in its result.


## **[NaLLM Graph Construction Demo](https://nallm-experiments.ew.r.appspot.com/?source=post_page-----877be33300a2--------------------------------)**

[nallm-experiments.ew.r.appspot.com](https://nallm-experiments.ew.r.appspot.com/?source=post_page-----877be33300a2--------------------------------)

If you are interested in learning more about generative AI and knowledge graphs, I would suggest taking a look at [Neo4j’s page about generative AI](https://neo4j.com/generativeai/).


# **Demonstration**

I tested the application by giving it the Wikipedia page for the [James Bond franchise](https://en.wikipedia.org/wiki/James_Bond) and inspected the generated knowledge graph.



<p id="gdcalert4" ><span style="color: red; font-weight: bold">>>>>>  gd2md-html alert: inline image link here (to images/image4.png). Store image on your image server and adjust path/filename/extension if necessary. </span><br>(<a href="#">Back to top</a>)(<a href="#gdcalert5">Next alert</a>)<br><span style="color: red; font-weight: bold">>>>>> </span></p>


![alt_text](images/image4.png "image_tooltip")


Example of the resulting graph

The provided graph subset showcases the generated graph, which, in my opinion, provides a reasonably accurate depiction of the Wikipedia article. The graph primarily consists of nodes representing books and individuals associated with those books, such as authors and publishers.

However, there are a few issues with the graph. For instance, Ian Fleming is labeled as a publisher rather than an author for most of the books he wrote. This discrepancy may be attributed to the difficulty the language model had in comprehending that particular aspect of the Wikipedia article.

Another problem is the inclusion of relationships between book nodes and the directors of films with the same titles, instead of creating separate nodes for the movies.

Finally, It’s worth noting that the LLM appears to be quite literal in its interpretation of relationships, as evidenced by using the relationship type “used” to connect the James Bond character with the cars he drove. This literal approach may stem from the article’s usage of the verb “used” rather than “drove.”

A full video of the demonstration can be found here:

Demo of KG Construction


# **Problems**

For a demonstration, this approach worked fairly well, and we think it shows that it’s possible to use LLMs to create knowledge graphs. However, we acknowledge certain issues need to be addressed within this approach:



* **Unpredictable output:** This is inherent to the nature of LLMs. We do not know how an LLM will format its results. Even if we ask it to output in a specific format, it might not obey. This might cause problems when trying to parse what it generates. We saw one instance of this while chunking the data: Most of the time, the LLM generated a simple list of nodes and edges, but sometimes the LLM would number the list. Tools to work around this are starting to be released, such as [Guardrails](https://shreyar.github.io/guardrails/) and [OpenAIs Function API](https://openai.com/blog/function-calling-and-other-api-updates). It’s still early in the world of LLM tooling, so we anticipate that this will not be a problem for long.
* **Speed**: This approach is slow and often takes several minutes for just a single reasonably large web page. There might be a fundamentally different approach that can make the extraction go faster.
* **Lack of accountability**: There is no way of knowing why the LLM decided to extract some information from the source documents or if the information even exists in the source. The data quality of the resulting knowledge graph is, therefore, much lower than the graph created by processes not leveraging LLMs.


# **Summary**

This blog post explored a use case of Large Language Models with Neo4j to extract insights from unstructured data by converting it into a structured representation in the form of a knowledge graph.

We discussed a three-step approach focusing on extracting nodes and relationships, entity disambiguation, and importing the data into Neo4j. By utilizing LLMs, anyone can automate the extraction process and efficiently process large amounts of unstructured data.

_However, there are challenges_ to address, including unpredictable output formatting, speed limitations, and the lack of accountability. Despite these issues, the combined power of LLMs and Neo4j offers a promising solution for unlocking the hidden value in unstructured data, even for non-technical users.

We hope that this blog post has provided you with valuable insights and practical knowledge to leverage LLMs and Neo4j in extracting knowledge from unstructured data. The code for this project can be found on [GitHub](https://github.com/neo4j/NaLLM), and if you want to know more about AI and Neo4j, I suggest taking a look at [Neo4j’s page about generative AI](https://neo4j.com/generativeai/).
