# Agentic RAG Demo 

This repo contains a python script where the Elastic python client is integrated with a streamlit UI - To showcase an example of agentic search using LangChain. 

To use this code directly, create a ```.env``` file and fill it with the following variables:

```
ELASTIC_ENDPOINT=<ELASTIC CLOUD ENDPOINT>
ELASTIC_API_KEY=<ELASTIC CLOUD API KEY>

# Enable custom search API
# https://developers.google.com/custom-search/v1/introduction/?apix=true
GCP_API_KEY=<GCP API KEY>
GCP_PSE_ID=<GCP PSE ID>


AZURE_OPENAI_SYSTEM_PROMPT="You are a helpful assistant. Be as concise and efficient as possible. Convey maximum meaning in fewest words possible."


AZURE_OPENAI_ENDPOINT=<AZURE ENDPOINT>
AZURE_OPENAI_API_VERSION=<AZURE API VERDSION>
AZURE_OPENAI_API_KEY=<AZURE API KEY>
AZURE_OPENAI_MODEL="gpt-4o-mini"
```

This code requires a connection to the Google Custom Search API and to an Elastic Cloud deployment. The agent's tools are API calls to either the Google CSAPI or to specific Elasticsearch indices - The latter is a semantic_search over indices containing PDF files processed using the `semantic_text` datatype. 

```
class ElasticSearcher:
    def __init__(self):
        self.client = Elasticsearch(
            os.environ.get("ELASTIC_ENDPOINT"),
            api_key=os.environ.get("ELASTIC_API_KEY")
        )
    
    def search(self, query, index="us_navy_dive_manual", size=10):
        response = self.client.search(
            index=index,
            body={
                "query": {
                    "semantic": {
                        "field": "semantic_content",
                        "query": query
                    }
                }     
            },
            size=size
        )
        return "\n".join([hit["_source"].get("body", "No Body") 
                            for hit in response["hits"]["hits"]])
```


This tutorial contains all the necessary steps for creating a semantic search index - I highly recommend checking it out! Once the indices are created, simplify modify the `elastic.search` call in `tools` to call the specific index name. 

```
tools = [
    Tool(
        name="WebSearch",
        func=lambda q: googler.search(q, n=3),
        description="Search the web for information. Use for current events or general knowledge or to complement with additional information."
    ),
    Tool(
        name="NavyDiveManual",
        func=lambda q: elastic.search(q, index="us_navy_dive_manual"),
        description="Search the Operations Dive Manual. Use for diving procedures, advanced or technical operational planning, resourcing, and technical information."
    ),
    Tool(
        name="DivingSafetyManual",
        func=lambda q: elastic.search(q, index="diving_safety_manual"),
        description="Search the Diving Safety Manual. Use for generic diving safety protocols and best practices."
    )
]
```