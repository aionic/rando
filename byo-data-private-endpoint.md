# Bring your own data private endpoint support for Azure Open AI

## Background

In the current public preview of bring your own data in OpenAI there is no support for leveraging private endpoints, this can be a blocker for secure/compliance focused customers that disallow leveraging public blogs
The bring your own data functionality in OpenAI is basically an abstraction/automation over the top of cognitive services.

## Workaround

### Identity

- Create a managed identity to be leveraged by the solution

### Storage

- Create a storage account
- Create a container
- Upload the files you want to leverage (supported .pdf, txt, markdown, docx, ppt)
- Grant access to the storage account to the managed identity created above
- Enable CORS Settings --> Resource sharing (CORS) [to-do tighten this up]
  - Set allowed origins *
  - Methods: Get, Options, Post, Put
  - Allowed headers *
  - Exposed headers *
  - Max age 0

### Cog Services

- Create cog services account
- Assign User identity from above identity step
- Go to networking and enable shared private access to connect cog services to storage
- Create "shared private access"  (rebranded private endpoint)

### Storage

- Accept the private endpoint connection
- Disable public network access

### Cog Services

Settings

- Enable Semantic Search (preview)

Search Management

- Create a data source [todo make this json]
  - Blob storage
  - Choose an existing connection (select the account)
  - Select the managed identity
  - Set the container

- Create an index
  - Add index json

```json
{

  "name": "test1234",
  "fields": [
    {
      "name": "id",
      "type": "Edm.String",
      "searchable": false,
      "filterable": true,
      "retrievable": true,
      "sortable": true,
      "facetable": false,
      "key": true,
      "synonymMaps": []
    },
    {
      "name": "content",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene",
      "synonymMaps": []
    },
    {
      "name": "filepath",
      "type": "Edm.String",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "synonymMaps": []
    },
    {
      "name": "title",
      "type": "Edm.String",
      "searchable": true,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "analyzer": "standard.lucene",
      "synonymMaps": []
    },
    {
      "name": "url",
      "type": "Edm.String",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "synonymMaps": []
    },
    {
      "name": "chunk_id",
      "type": "Edm.String",
      "searchable": false,
      "filterable": false,
      "retrievable": true,
      "sortable": false,
      "facetable": false,
      "key": false,
      "synonymMaps": []
    }
  ],
  "scoringProfiles": [],
  "suggesters": [],
  "analyzers": [],
  "normalizers": [],
  "tokenizers": [],
  "tokenFilters": [],
  "charFilters": [],
  "semantic": {
    "defaultConfiguration": null,
    "configurations": [
      {
        "name": "test",
        "prioritizedFields": {
          "titleField": {
            "fieldName": "title"
          },
          "prioritizedContentFields": [
            {
              "fieldName": "content"
            }
          ],
          "prioritizedKeywordsFields": []
        }
      }
    ]
  }
}
```

- Add an Indexer
  - Name: Anything you want
  - Index: the index created above
  - Data source: the data source you listed up ahead
  - Schedule (adjust based on how often new data is added)

### Open AI

- Deploy an Open AI Workspace
- Create a chat-gpt deployment
- Data Source
  - Data Source: Azure Cognitive search
  - Subscription: Sub where cog services is hosted
  - Azure Cognitive Search Service: Serviced named above
  - Cognitive Search Index: Index named above
- Data Field Mapping:
  - Content data: content
  - File name: field where your doc lives
  - Title: title
  - URL: blank
- Data Management
  - Select the semantic search config and acknowledge
- Save and close

### Testing

- Test by chatting with your data in the studio
