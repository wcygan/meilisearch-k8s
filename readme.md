# Meilisearch on Kubernetes

[Reference](https://github.com/meilisearch/meilisearch-kubernetes)

## Quick Start

```bash
# Deploy Meilisearch
skaffold run

# Check the status of the pods
 kubectl get pods -n meilisearch 
NAME            READY   STATUS    RESTARTS   AGE
meilisearch-0   1/1     Running   0          28h
```

## Data Modeling

Here's a potential data model design for a "Search" feature on a website:

```proto
syntax = "proto3";

package search.v1;

// Define the basic message for each content type.
message Post {
  string id = 1;
  string title = 2;
  string body = 3;
  string author_id = 4;
  string created_at = 5;
  int32 likes = 6;
}

message User {
  string id = 1;
  string username = 2;
  string display_name = 3;
  string bio = 4;
  string joined_at = 5;
}

message Comment {
  string id = 1;
  string post_id = 2;
  string author_id = 3;
  string text = 4;
  string created_at = 5;
}

// A "SearchResult" can be any of the above
message SearchResult {
  oneof document {
    Post post = 1;
    User user = 2;
    Comment comment = 3;
  }
}

// The request for searching (for a typeahead or general search)
message SearchRequest {
  string query = 1;
  // You could also include optional filters, pagination, etc.
}

// The response with a list of results
message SearchResponse {
  repeated SearchResult results = 1;
}

// The service definition
service SearchService {
  rpc Search(SearchRequest) returns (SearchResponse);
}
```

With this design, the application might be structured like this:

```bash
┌───────────────┐
│   Frontend    │
│ (UI / Client) │
└──────┬────────┘
       │  (1) ConnectRPC + Protobuf
       │
┌──────▼────────┐
│    Backend    │
│ (Search Svc)  │
└──────┬────────┘
       │  (2) REST + JSON
       │
┌──────▼────────┐
│ Meilisearch   │
└───────────────┘
```

The search service would do some translation between the protobuf messages and the REST API of Meilisearch so that the frontend can use the protobuf definitions while still leveraging the capabilities of Meilisearch.