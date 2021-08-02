# Information Retrieval(Boolean AND based search queries)

In this project, I build a simple Information Retrieval system based on Boolean AND operation. The main idea is to create a fast retrieval system using inverse indexing and create relevant posting lists. I also explore various optimization methods to enhance the overall operation.

## 1. Toolkit 
The first part involves loading the dataset. The nltk package is used for stemming and lemmatization of tokens.

The first function ```directory_list``` takes in the argument ```datasetname``` and searches for the corresponding folder name. It returns the absolute path of the dataset, ```fullpath``` with a list of ```filenames, docfilename```. In this project, I used the dataset \"HillaryEmails" for analysis and testing. The function searches for the dataset and returns the absolute path to the variable ```path``` and the file listing to ```docfilename```.

## 2. Tokenization
The tokenization process takes place in the text and the name of the file and returns a list of token and document name pair. 
The function ```tokenizer``` reads in the path and the ```docfilename``` list to collect the text documents to process. Documents are first stripped of string literals such as tabs, newlines and empty spaces so that the empty tokens will not produced. The output from tokenization returns all the tokens in a document.

## 3. Linguistic Modules
The function ```processString``` performs simple linguistic transformations to an input string by removing punctuations, stop words, lowering character case and stemming. These steps reduce memory usage. The ```Snowball``` stemmer was implemented due to its balance between speed and size reduction.

Removing punctuations and stop words are essential as these carry little to no meaning, and reduces the number of redundant token terms in the dictionary. Similarly, different capitalizations may produce multiple tokens. Words conforming to lowercase requirements could save space in the dictionary.

Stemming and lemmatization reduces words into their base forms, and prevents similar words from appearing in the dictionary. However, stemming may not produce actual words while lemmatizers do. While it is not required, both lemmatizers and various stemmers are added for the bench-marking tests.  

## 4. Token Sorting
The ```sort_token``` function sorts the token terms in alphabetical order and subsequently sorts them by the inverted index key values.

Figure below illustrates the database structure of the retrieval indexing system using a dictionary of key-value pairs. In the dictionary, the token represents the key while the indices represent the document ID value for each pair. Document names can be found using the indices from the filename array. For example, the token term ```fish``` has a posting list of {2,5,6,7,8}. These indices refer to the inverted index of the filename array containing the document filename. The documents can subsequently be retrieved. In this illustration, the indices are sorted in numerical order within the posting lists.
![Alttext](https://user-images.githubusercontent.com/71308636/127807845-a5819120-579c-4c72-b575-513cb843ecda.png)

In order to obtain the dictionary and posting lists, token terms need to be further processed. Figure following this paragraph illustrates the process of sorting the token terms and the forming of the posting lists. The token terms with its indexes are firstly collected from the documents that are previously read into memory. The token terms are then placed in a dictionary as keys. If the key-index pair exists in the dictionary, no modification is required. Else, the corresponding key-index pair is added. The dictionary is initially sorted by keys in alphabetical order, followed by the indexes in numerical order. Then, the posting lists are extended in order for each key and this results in a complete database of terms and its corresponding posting lists when the collection has been completely scanned.
![image](https://user-images.githubusercontent.com/71308636/127808479-990ed864-5fcb-44a4-9113-c37be5c58827.png)

## 5. Postings Lists Transformation
The merge posting function merges two lists together by having pointers at the head of each list then comparing the values of all indexes in the two lists. If both pointer values are equal, the value is added to the posting list and both pointers are shifted to the next index. Else, the pointer with the smaller value will shift by one index. The ```intersect``` function extracts two posting lists from the dictionary corresponding to the two words provided. It produces the merged posting list for the two words.

## 6. Merging Postings List
The ```mergeposting``` function takes in two lists, ```list1``` and ```list2``` as the arguments, creates an empty target posting list and initializes the pointers, ```i``` and ```j```. These pointers ```i``` and ```j``` point to the head of each list and is used for comparison. If they are equal, both pointers are incremented. Otherwise, only the pointer with the lower value will increment. This is repeated until one pointer reaches the end.

## 7. Information Retrieval System
The ```retrieval``` function takes in the query string, processes it using the ```processString``` function and splits it into words. It pairs the words and uses the merge listing algorithm to produce a merged posting list. An example on query processing is shown in the figure below. The terms in the query after processing become tokens. Each token term contains a posting list if it exists in the dictionary. Two posting lists are merged together at a time. The resultant posting list will then be merged with the posting list of the next token term. The process is repeated until all terms have been considered.

![image](https://user-images.githubusercontent.com/71308636/127808977-5a9cce10-296b-4976-a167-8928ac6db6c7.png)
![image](https://user-images.githubusercontent.com/71308636/127808983-374519c2-0681-4121-827c-45f33f451d80.png)

## 8. Optimization
Three optimizations were further added to the system, namely, Short-to-Long postings List merge, skip pointers, and dictionary based compression techniques. The first two methods reduces query answering time, and the third serves the purpose of using disks as memory, thus improving the scalability of an in-memory index construction. The three methods will be described separately as follows.

### 8.1 Short-to-Long Postings List (STPL)
This method aims to reduce response time for query processing. The insight behind is that, given a Boolean AND query, switching the execution order among query terms could avoid unnecessary comparisons, and accelerate query processing. Instead of naively intersecting the query terms based on their order, we exploit the document frequency for the query terms that show up in the collection. We do so by sorting the query terms by ascending frequency. In this way, the length of effective postings list can be greatly shortened, avoiding unnecessary intersecting and speeding up the processing.

### 8.2 Skip Pointers
Skip pointers is another method targeted at accelerating query processing. Different from STPL, which effects before executing intersecting, skip pointers improves the intersecting process. When conducting list merging, the baseline algorithm walks through the two postings simultaneously, and the time complexity is the summation of the length of the posting lists. To improve efficiency, we augment postings merging with skip pointers. This reduces the number of comparisons while maintaining the same search result. 

![image](https://user-images.githubusercontent.com/71308636/127809259-3099142c-5c43-44e6-b56d-551f3ccce0a9.png)

Figure given above illustrates an example on skip pointer processing using *Brutus* AND *Caesar* as the query. After tokenization and indexing, the postings list of the *Brutus* and *Caesar* is as shown in the figure. 
We advance through the lists as usual until we obtain *8*, and move it to the results list. Both pointers shift such that the pointer points to *16* on the upper list and *41* on the lower list. The smaller value between both pointers is *16*. Since the value in the upper list is smaller, we move three positions to *28* and check if the value in the upper list is less than the value in the lower list. Since *28* is less than *41*, we can use the skip list pointer and move the upper pointer to *28*. In this way, we save two comparison operations by skipping *19* and *23*.

We added two skip pointers, one for each postings list. During the operation, the skip pointers move with single step size, and searches for the item to stop on and make comparisons. The comparison operation remains the same. 

### 8.3 Dictionary Compression
The main reason for compressing dictionaries is to decrease the frequency of access on the disk. With a smaller footprint, it can be fitted into memory. A large portion of the dictionary in memory also improves the query throughput. As the number of documents in the collection increases, the size of dictionary also increases. In addition to storage reduction, a better decompression algorithm can encourage more efficient use of the cache which improves the rate of data transfer from disk to memory. This balances the trade-off between space and speed.
Here we mention two techniques:
#### 8.3.1 Dictionary As A String (DAAS)
The simplest way of storing vocabulary is in an array of fixed width (20 characters) along with pointers to its posting list. For large collections, the memory usage is large. Using a fixed width  wastes space for shorter words and terms longer than the fixed width array cell cannot be stored.
These shortcomings can be overcome by storing terms of the dictionary as a long string of characters with pointers to every term to demarcate their beginning in the string. This scheme stores every term in an average of 8 bytes, saving 60% of the storage as compared to using 20 bytes while allowing the term pointers to be stored, as shown in Figure below: 
![image](https://user-images.githubusercontent.com/71308636/127809844-ca23b146-6fe6-4ad3-a6f0-3e1017c7ba67.png)


#### 8.3.2 Blocked Storage of Dictionary As A String (BS-DAAS)
Another improvement to the above-mentioned technique is to store terms in blocks of a pre-defined size *k* wherein the pointer points to the beginning of every block. In this manner, an additional *k* bytes would be required to store the term length but *k-1* term pointers can be removed. As a result, increasing the block size *k* achieves better compression. However, there is a trade-off between the computation of the compression algorithm and the speed of term lookup. In a binary search algorithm, we would first need to find the block to which the term belongs to and find the term within the block, whereas without compression we just apply binary search on the dictionary to find the term.
![image](https://user-images.githubusercontent.com/71308636/127809942-33c98f0a-6f00-469a-85f6-1e3c6ced3e63.png)

The motivation of string compression is to explore similarities in strings to remove redundancies. In a dictionary, common prefixes exists among consecutive entries in an alphabetically sorted list. Special characters can be used to represent the common prefix of the term list. Therefore, terms can be further reduced in size. This is also known as front coding.

However, given the amount of data in this era, it might not be feasible to store the entire dictionary in memory for enormous collections or on hardware having limited memory even when using the best compression methods. Having said that, the techniques discussed and implemented are very space efficient. Both techniques are lossless, however there are other techniques which result in better compression ratios and speed but are lossy.
