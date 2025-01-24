# **Documentation - RAG (Retrieval-Augmented Generation)**

## **Objective**
This project demonstrates the implementation of a Retrieval-Augmented Generation (RAG) system for information retrieval based on text. RAG combines the capabilities of language models (LLMs) with specific data retrieved from a vector database. 

### **Advantages**
- **Reduces the amount of data sent to the LLM**: Only the relevant data is sent to the model, optimizing bandwidth and processing time.
- **No need for fine-tuning**: The model doesn't require additional training, as semantic retrieval provides tailored context.
- **Greater control and personalization**: Enables seamless integration with structured and unstructured documents.
- **Semantic search using embeddings**: Ensures searches are based on meaning rather than exact keywords.

---

## **Architecture**
The project uses the following tools and technologies:
1. **Docker**: Used to handle Python version compatibility and package isolation.
2. MRPT:
3. **LangChain**: Divides texts into smaller chunks (chunking) and manages text-processing workflows.
4. **Vectordb2**: A vector memory library to store and retrieve divided text sections.
5. **Rust**: Installed to compile dependencies for `vectordb2`.

---

## **Installation Steps**

### **1. Setting up the Environment**
Some libraries have strict Python version requirements. To avoid conflicts, we used Docker to isolate the necessary packages.

#### Installation MRPT via Docker:
```bash
git clone https://github.com/vioshyvo/mrpt.git
cd mrpt
docker build -t mrpt .
docker run --rm -it mrpt````

#### Installing additional dependencies:

- **Installing Rust (for `vectordb2` dependencies)**:

````bash
`curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh source "$HOME/.cargo/env"`
````

- **Installing Python libraries**:
````bash
`pip install langchain-core langchain vectordb2`
````
---

## **Text Processing**

The input text was extracted from a Markdown file hosted in an online repository. The extraction and preprocessing steps are as follows:

### **1. Extracting raw text**

````python
`import requests bruto_text = requests.get('https://raw.githubusercontent.com/abjur/constituicao/main/CONSTITUICAO.md').text`
````
### **2. Splitting the text into sections**

#### Using Regex:

````
`import re  pattern = r'^##\s+(.*)$' sections = re.split(pattern, bruto_text, flags=re.MULTILINE) sections = [section.strip() for section in sections[1:]]`
````
#### Using LangChain:

````
`from langchain.text_splitter import MarkdownHeaderTextSplitter  headers = [("##", "Chapter")] markdown_splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers) sections = markdown_splitter.split_text(bruto_text)`
````
---

## **Using Memory (Vectordb2)**

### **1. Configuring vector memory**

We used `vectordb` to create a vector memory for storing text sections. The following configuration specifies how text should be chunked:

````
`from vectordb import Memory  memory = Memory(chunking_strategy={"mode": "sliding_window", "window_size": 128, "overlap": 8})`
````
### **2. Adjusting chunk size and overlap**

Chunk size and overlap were adjusted to ensure better text segmentation. The following example demonstrates this:

````python
from langchain.text_splitter import RecursiveCharacterTextSplitter  

chunk_size = 20 
chunk_overlap = 3  

text = "The Federative Republic of Brazil, formed by the indissoluble union of the States, Municipalities, and the Federal District"  

splitter = RecursiveCharacterTextSplitter(chunk_size=chunk_size, chunk_overlap=chunk_overlap) 

chunks = splitter.split_text(text)  

for chunk in chunks:     
print(chunk)`
````
Here, the text is divided into 20-character chunks with a 3-character overlap between each chunk.

### **3. Saving data to vector memory**

After splitting the text into sections, each section is saved to the vector memory along with descriptive metadata (e.g., chapter and source of the text):
````python
for i in range(0, len(sections)):     
chapter = sections[i].metadata  # Retrieves the chapter title     
content = sections[i].page_content  # Retrieves the section content     

# Metadata for enriching the stored data     
metadata = {'chapter': chapter, 'source': 'Federal Constitution'}      
# Saves the text and metadata in the vector memory     

memory.save(content, metadata)`
````
#### **Detailed Explanation:**

1. **`sections[i].metadata`**: Retrieves the chapter title (e.g., "Chapter 1").
2. **`sections[i].page_content`**: Retrieves the content of the corresponding section.
3. **Metadata**: Additional information is added for better contextualization during retrieval.
4. **`memory.save()`**: Saves the section text and metadata into the vector memory.

### **4. Performing semantic searches**

Semantic searches use embeddings to find text sections that are semantically similar to a query:

````python
`memory.search('indigenous', top_n=5)`

#### **Explanation:**

- **`memory.search()`**: Searches the vector memory for relevant sections.
- **`top_n=5`**: Returns the top 5 most relevant matches.
- The search relies on semantic similarity rather than exact keyword matches, making it more robust.
````
---

## **Expected Results**

This system achieves the following:

1. Splits long texts into smaller, manageable chunks for efficient analysis.
2. Stores text sections in a vector memory with descriptive metadata.
3. Enables semantic information retrieval, returning the most relevant sections based on meaning.
