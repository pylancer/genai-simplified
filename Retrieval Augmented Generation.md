# genai-rag
Retrieval-augmented generation (RAG) is a hybrid framework that integrates retrieval and generative models to produce contextually accurate and information-rich text. It's like a supercharged researcher and writer duo.

```mermaid
 flowchart LR
subgraph Ingestion["Ingestion"]
        Index[("Index")]
        Embeddings["Embeddings for retrieval tasks"]
        Chunks((("Chunks")))
        Documents>"Documents"]
  end
 subgraph Retrieval["Retrieval"]
        Top_K["Top K retrieved chunks"]
        Query["Query"]
  end
 subgraph Synthesis["Synthesis"]
        Response["Response"]
        LLM["LLM"]
  end
 subgraph BasicRAGPipeline["BasicRAGPipeline"]
        Ingestion
        Retrieval
        Synthesis
  end
  Documents --> Chunks --> Embeddings --> Index --> Top_K --> Synthesis
  Query --> Index
  Query --> Synthesis
  LLM --> Response
```


Here's a simplified explanation of the RAG pipeline:

1. **Document Ingestion**: This process occurs offline. The retriever component of RAG searches through a vast corpus of text to find relevant documents.

2. **Online Query**: When an online query comes in, the retrieval of relevant documents and generating a response occurs.

3. **Retrieval of Relevant Documents**: The retriever component of RAG finds relevant documents that answer the query.

4. **Generation of Response**: The generator component uses these documents and the original query to generate a detailed and up-to-date answer.

RAG has several advantages:

- **Empowering LLM solutions with real-time data access**: AI solutions that use Large Language Models (LLMs) can remain up-to-date and current with RAG, facilitating direct access to additional data resources.
- **Preserving data privacy**: Ensuring data privacy is crucial for enterprises. With a self-hosted LLM, sensitive data can be kept on-premises.
- **Mitigating LLM hallucinations**: RAG reduces the likelihood of hallucinations by providing the LLM with relevant and factual information.

The RAG Triad evaluates Retrieval-Augmented Generation (RAG) applications to ensure they are free from hallucinations. It consists of three evaluations:

1. **Context Relevance**: This evaluates the relevance of each chunk of context to the input query. It's crucial because the context will be used by the Large Language Model (LLM) to form an answer. Any irrelevant information in the context could be woven into a hallucination.

2. **Groundedness**: This evaluates whether the context supports the response. LLMs often stray from the facts provided, exaggerating or expanding to a correct-sounding answer.

3. **Answer Relevance**: This evaluates the relevance of the final response to the user input.

By reaching satisfactory evaluations for this triad, we can make a nuanced statement about our application’s correctness; it is verified to be hallucination-free up to the limit of its knowledge base. In other words, if the vector database contains only accurate information, then the answers provided by the RAG are also accurate.



**Sentence Window Retrieval**

Sentence Window Retrieval (SWR) is an advanced technique in Retrieval-Augmented Generation (RAG) systems. Here's a simplified explanation:

1. **Sentence Embedding**: Each sentence in a document is embedded separately. This provides excellent accuracy for the query-to-context cosine distance search.

2. **Context Extension**: After fetching the most relevant single sentence, the context window is extended by a certain number of sentences (k) before and after the most pertinent sentence. This broader context is used for a more grounded generation.

The philosophy behind SWR is to retrieve a pointed context from the custom knowledge base based on the query and use a broader version for a more grounded generation. In this process, a limited set of sentences is embedded for retrieval². This approach enhances the solution’s ability to identify similarities in requests, incorporating confirmed responses as additional context to mitigate hallucinations and ensure consistent service quality².


**Retrieval Augmented Generation (RAG)** is an advanced artificial intelligence technique that combines information retrieval with text generation. It allows AI models to retrieve relevant information from a knowledge source and incorporate it into generated text1. Let’s delve into the specifics of the Auto-merging Retriever, also known as the Parent Document Retriever.

Naive RAG:

The starting point of the RAG pipeline involves a corpus of text documents. We’ll skip everything before this point, leaving it to the open-source data loaders that can connect to various sources, from YouTube to Notion.
In the Naive RAG approach, we retrieve relevant information from the corpus and use it as context for a Large Language Model (LLM) to generate answers.
However, Naive RAG has limitations, especially when dealing with long documents or complex queries.

Auto-merging Retriever:

The idea behind this technique is similar to the Sentence Window Retriever. We search for more granular information (child chunks) within documents.
The Auto-merging Retriever goes a step further. It recursively “merges” subsets of leaf nodes that reference a parent node beyond a given threshold.
It extends the context window by combining related chunks from different parts of the document.
This larger context is fed to an LLM for reasoning and text generation23.
In summary, Auto-merging Retriever enhances the context provided to the language model by consolidating potentially disparate, smaller contexts into a more comprehensive one. This approach aims to improve the quality of generated responses by incorporating relevant information from the retrieved documents23. 🤖📚

**Evaluate & Iterate**
- Iterate with different hierarchical structures (number of levels, children) and chunk sizes
- Evaluate app versions with the RAG Triad
- Track experiments to pick the best structure
- Gain intuition about hyperparameters that work best with certain doc types 
- Auto-merging is complimentary to sentence-window retrieval

```mermaid
flowchart TD
 subgraph Context_Relevance["Context_Relevance"]
        context_selection["context_selection"]
        TruLlama{{"TruLlama"}}
        f_qs_relevance["f_qs_relevance"]
  end
 subgraph Answer_Relevance["Answer_Relevance"]
        f_qa_relevance["f_qa_relevance"]
  end
 subgraph Groundedness["Groundedness"]
        feedback_Groundedness{{"feedback.Groundedness"}}
        f_groundedness["f_groundedness"]
        grounded["grounded"]
  end
 subgraph RAG_Triad["RAG_Triad"]
        Groundedness
        Context_Relevance
        Answer_Relevance
        Feedback{{"Feedback"}}
  end

 subgraph build_sentence_window_index["build_sentence_window_index"]
        SentenceWindowNodeParser{{"SentenceWindowNodeParser"}}
        sentence_context["sentence_context"]
  end
 subgraph get_sentence_window_query_engine["get_sentence_window_query_engine"]
        postproc["postproc"]
        MetadataReplacementPostProcessor{{"MetadataReplacementPostProcessor"}}
        rerank["rerank"]
        SentenceTransformerRerank{{"SentenceTransformerRerank"}}
        postproc & rerank --> as_query_engine
  end
 subgraph SentenceWindowRetreval["SentenceWindowRetreval"]
        build_sentence_window_index
        query_sw["query_sw"]
        get_sentence_window_query_engine
  end
          window_response["window_response"]

 subgraph build_automerging_index["build_automerging_index"]
        HierarchicalNodeParser{{"HierarchicalNodeParser"}}
        mergingContext["mergingContext"]
  end
 subgraph get_automerging_query_engine["get_automerging_query_engine"]
        AutoMergingRetriever{{"AutoMergingRetriever"}} --> retriever
        SentenceTransformerRank_merge --> rerank_merge
        retriever & rerank_merge --> RetrieverQueryEngine
        
  end
 subgraph AutoMergingRetrieval["AutoMergingRetrieval"]
        build_automerging_index
        query_am["query_am"]
        get_automerging_query_engine
  end
          auto_merging_response["auto_merging_response"]

 subgraph TruLens_Evaluation["TruLens_Evaluation"]
        provider["provider"]
        fOpenAI["fOpenAI"]
        eval_questions["eval_questions"]
        tru_recorder["tru_recorder"]
        eval_query["eval_query"]
        records["records"]
        feedback["feedback"]
        RAG_Triad
        query_engine_eval["query_engine_eval"]
            reset_database{{"reset_database"}}
                    get_leaderboard{{"get_leaderboard"}}
                            run_dashboard{{"run_dashboard"}}



  end
 subgraph Evaluation["Evaluation"]
  end
    MetadataReplacementPostProcessor --> postproc
    SentenceTransformerRerank --> rerank
    VectorStoreIndex["VectorStoreIndex"] -- automerging_index ---> get_automerging_query_engine
    VectorStoreIndex -- sentence_index --> get_sentence_window_query_engine
    f_groundedness --> tru_recorder
    f_qa_relevance --> tru_recorder
    f_qs_relevance --> tru_recorder
    grounded -- groundedness_measure_with_cot_reasons --> f_groundedness
    provider --> feedback_Groundedness & Feedback
    feedback_Groundedness --> grounded
    Feedback -- qs_relevance --> f_qs_relevance
    context_selection -.-> f_qs_relevance
    TruLlama -- select_source_nodes --> context_selection
    Feedback -- relevance_with_cot_reasons --> f_qa_relevance
    fOpenAI --> provider
    RetrieverQueryEngine -- automerging_query_engine --> query_am
    query_am --> auto_merging_response & TruLens_Evaluation
    as_query_engine -- sentence_window_engine --> query_sw
    query_sw --> window_response & TruLens_Evaluation
    ServiceContext["ServiceContext"] -- embedmodel-bge_small_en_v15 --> service_context["service_context"]
    mergingContext --> VectorStoreIndex
    sentence_context --> VectorStoreIndex
    service_context <-.-> mergingContext & sentence_context
    input_file["input_file"] --> SimpleDirectoryReader{{"SimpleDirectoryReader"}}
    SimpleDirectoryReader -- documents --> Document{{"Document"}}
    Document -- document --> decision{"Sentence Window or Automerging"}
    LLM["LLM"] --> decision
    decision --> SentenceWindowNodeParser & HierarchicalNodeParser
    OpenAI{{"OpenAI"}} -- gpt_35_turbo --> LLM
    SentenceWindowNodeParser -- node_parser.get_nodes_from_documents --> ServiceContext
    Groundedness --> run_dashboard
    Context_Relevance --> run_dashboard
    Answer_Relevance --> run_dashboard
    run_dashboard --> get_leaderboard
    HierarchicalNodeParser -- leaf_nodes --> ServiceContext
    query_engine_eval -- get_prebuilt_trulens_recorder --> tru_recorder
    eval_questions -- question --> query_engine_eval
    tru_recorder --> eval_query
    eval_query -- get_records_and_feedback --> records & feedback
      llamaindex{{"llamaindex"}}
      trulens_eval{{"trulens_eval"}}
    style TruLlama fill:#AA00FF
    style feedback_Groundedness fill:#AA00FF
    style Feedback fill:#AA00FF
    style run_dashboard fill:#AA00FF
    style get_leaderboard fill:#AA00FF
    style reset_database fill:#AA00FF
    style SentenceWindowNodeParser fill:#2962FF
    style MetadataReplacementPostProcessor fill:#2962FF
    style SentenceTransformerRerank fill:#2962FF
    style HierarchicalNodeParser fill:#2962FF
    style AutoMergingRetriever fill:#2962FF
    style VectorStoreIndex fill:#2962FF
    style ServiceContext fill:#2962FF
    style SimpleDirectoryReader fill:#2962FF
    style Document fill:#2962FF
    style OpenAI fill:#D50000
    style llamaindex fill:#2962FF
    style trulens_eval fill:#AA00FF
```
**Sources:**
https://learn.deeplearning.ai/building-evaluating-advanced-rag/lesson/1/introduction
RAG:
(1) Retrieval Augmented Generation (RAG): A Comprehensive Guide. https://www.datastax.com/guides/what-is-retrieval-augmented-generation.
(2) Understanding Retrieval-Augmented Generation: A Simple Guide. https://medium.com/@amodwrites/understanding-retrieval-augmented-generation-a-simple-guide-d638ac92c123.
(3) RAG 101: Demystifying Retrieval-Augmented Generation Pipelines. https://developer.nvidia.com/blog/rag-101-demystifying-retrieval-augmented-generation-pipelines/.
(4) RAG 101: Demystifying Retrieval-Augmented Generation Pipelines. https://developer.nvidia.com/blog/rag-101-demystifying-retrieval-augmented-generation-pipelines/.
(5) Understanding Retrieval-Augmented Generation: A Simple Guide. https://medium.com/@amodwrites/understanding-retrieval-augmented-generation-a-simple-guide-d638ac92c123.
(6) RAG Pipelines From Scratch | Haystack. https://haystack.deepset.ai/blog/rag-pipelines-from-scratch.
RAG Triad:
(1) RAG Triad - TruLens. https://www.trulens.org/trulens_eval/core_concepts_rag_triad/.
(2) How to Evaluate Retrieval Augmented Generation (RAG) Applications .... https://zilliz.com/blog/how-to-evaluate-retrieval-augmented-generation-rag-applications.
(3) Evaluating Multi-modal RAGs. https://web.stanford.edu/class/cs329t/slides/lecture_16_2.pdf.
(4) undefined. https://docs.ragas.io/en/latest/concepts/testset_generation.html.
Sentence Window Retrieval:
(1) Advanced RAG Techniques: an Illustrated Overview. https://pub.towardsai.net/advanced-rag-techniques-an-illustrated-overview-04d193d8fec6.
(2) A Guide to Advanced RAG Techniques for Success in Business Landscape. https://blogs.sap.com/2023/12/22/a-guide-to-advanced-rag-techniques-for-success-in-business-landscape/.
