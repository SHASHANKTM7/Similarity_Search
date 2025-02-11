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

