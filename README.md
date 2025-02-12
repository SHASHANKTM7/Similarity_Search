# Similarity Search
- Similarity Search Basically Means Retrival Of Documents From The Vector Database.

## What Was Performed?
- Multiple Links Where Extracted From The Target Link
- Considered Each Link And Obtained Information In Form Of Summary which where considered as Document for the corresponding Link.
- Documents where converted to Embeddings.
- Initialized the faiss index and stored the embeddings in it.
- Provided Query was converted to embeddings
- Query Embeddings was passed to the faiss index to search for related Document Embedding
- Got Distances And Indices as Output 
- Mapped and obtained the related Document ID and Document

## What Is Faiss?
- It is vector database used to store data and it has library for efficient similarity search and clustering of dense vectors.
- **IndexFlatL2** is one of the indexing methods that uses  L2 distance metric for nearest neighbor searches.


## Why Was Faiss Vector Database Considered?
- Faiss enables Faster Retrival due to its algorithms. Some of them are:-
  -  **Product Quantization** : this method is responsible for obtaining compressed representations of high dimensional vectors.
  -  **Kmeans Clusturing** : this method clusturs data to narrow down search space .helps in identifying relavent clusturs when query is provided.
 
## Why Is FAISS IndexFlat Needed?
- Stores embeddings efficiently for fast retrieval.
- Optimizes similarity search (e.g., L2 distance, cosine similarity).

## Steps Involved To Achive The Task
- Imported All the Nessesery Libraries.( Here I used Scrapy to scrap the data).

- Loaded the Summarization Model (t5 model was used).
- I have created a class  and two methods inside it.
   - The class is inheriting from scrapy.Spider.
   - The Target Link is provided here along with the name of the spider.
   - The first method is parse() which is resposible for processing of initial response recieved from the target URL.and also extracting href atttributes from anchor tags.(later only important links such as http is chosen 
   - The second method exracts text content from page and generates summary for each extraction with the help of T5 model.
- To make the above steps work the following below steps are essential:-
  - creating scrapy process that runs in jupyter notebook
  - providing class name so that it guides the spider to crawl
  - starting the process or runs spider and collects data.
  
  ```python
  scraped_data = []
  class LinksSpider(scrapy.Spider):
    name = "link_spider"
    start_urls = ["https://www.langchain.com"]  # Replace with your target link

    def parse(self, response):
        # Extract all links from the main page
        links = response.css("a::attr(href)").getall()
        for link in links:
            if link.startswith("http"):  # Ensure it's a valid absolute URL
                yield response.follow(link, callback=self.parse_link)

    def parse_link(self, response:
        # Extract text content from the page
        text_content = " ".join(response.css("p::text").getall())  # Extract all paragraph text

        if text_content:
            # Generate a summary (max length 100 words)
            summary = summarizer(text_content, max_length=100, min_length=30, do_sample=False)[0]["summary_text"]
        else:
            summary = "No summary available"

        # Store extracted data in a list
        scraped_data.append({"url": response.url,
            "title": response.css("title::text").get(),"summary": summary})  
  ```


## Creating dataframe  
- Creating DataFrame and storing the collected Documents along with document ID created.
- The next step is loading sentence transformer  model called mini LM.    
- And with the help of this model convert the documents to embeddings.
- Initialize the faiss index and pass the dimension parameter

## Retrieving Related Documents Through Index Using L2 Distance For Obtaining Distance
-  Create Function and pass query as argument and  convert it to embeddings using the same mini LM.
-  Search it in index along with given paramater top_k.
-  Distances and indices can be obtained from it.
```python
def search_query(query, top_k=3):
    # Convert query into embedding
    query_embedding = model.encode([query])
    
    # Search the FAISS index for relevant documents
    distances, indices = index.search(np.array(query_embedding), top_k)
    
    # Return matching documents and corresponding IDs
    results = []
    for i in range(top_k):
        doc_index = indices[0][i]  # Get index from FAISS results
        results.append({
            'ID': int(data_frame.iloc[doc_index]['doc_ID']),  # Use index to get the corresponding ID
            'document': data_frame.iloc[doc_index]['DOCUMENTS'],  # Use index to get the document
            'similarity_score': float(distances[0][i]) # Similarity score from FAISS
        })
    
    return results,indices
```


## Retrieving Related Documents Through Index Where Cosine Similarity Search Is Performed
- Normalize embeddings before passing to faiss.
- Used IndexFlatIP instead for initializing and passing the dimension
- Stored the normalised embedding in index and obtained distances and indicies
```python
def search_query(query, top_k=3):
    query_embedding = model.encode([query])
    faiss.normalize_L2(query_embedding)  # Normalize query for cosine similarity

    distances, indices = index.search(np.array(query_embedding), top_k)

    results = []
    for i in range(top_k):
        doc_index = indices[0][i]
        results.append({
            'ID': int(data_frame.iloc[doc_index]['doc_ID']),
            'document': data_frame.iloc[doc_index]['DOCUMENTS'],
            'similarity_score': float(distances[0][i])  # Now it's cosine similarity
        })
    
    return results
```

