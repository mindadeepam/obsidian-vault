
Fundamentals for any POC seems to be the following:

1. get embeddings using as basic as tfidf/sentence transformers.
2. use faiss to index those embeddings and get fast Approximate Neighbour Search over those embeddings.


```python

import faiss
item_from_id = "item with id, probably a df"
embedding_to_item_mapping = "some_mapping to map dense vectors to items/original_text, maybe embedding_array with same id/order"
dense_vectors = '<your dense embeddings matrix, in this case embeddings of arxiv paper abstracts>'

dimension = dense_vectors.shape[1]
index = faiss.IndexFlatIP(dimension)  # Inner product is equivalent to cosine similarity for normalized vectors
faiss.normalize_L2(dense_vectors)  # Normalize vectors
index.add(dense_vectors)

def get_similar_vectors(query_vector, top_n=5):

    query_vector = query_vector.reshape(1, -1)
    faiss.normalize_L2(query_vector)
    distances, indices = index.search(query_vector, top_n + 1)  # +1 because it will include the query paper itself
    return distances, indices

# Example usage

query_vector = "..." # get the query vector using input_text/item_id/etc
distances, indices = get_similar_vectors(query_vector)
print(item_from_id[indices])
```


