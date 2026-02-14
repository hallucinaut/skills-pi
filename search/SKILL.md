---
name: search
description: "Implement powerful full-text search with Elasticsearch, including indexing, querying, aggregations, and relevance tuning for production applications."
---

# Search Skill

Implement powerful full-text search capabilities with Elasticsearch for production applications.

## When to Use

Use this skill when the user wants to:
- Implement full-text search functionality
- Build search engines for applications
- Implement autocomplete and suggestions
- Create faceted search with filters
- Implement geospatial search
- Build analytics and log analysis systems
- Implement fuzzy matching and typo tolerance
- Create real-time search indexes
- Implement multi-language search
- Build recommendation engines
- Implement complex query DSL
- Set up search relevance tuning

## Technology Stack

### Core Technologies

#### Elasticsearch
- **Distributed search and analytics engine**
  - Full-text search
  - Structured queries
  - Aggregations and analytics
  - Geospatial search
  - Real-time indexing
  - Horizontal scalability

### Client Libraries

#### Python
- **elasticsearch**: Official Python client
- **elasticsearch-dsl**: High-level library for queries
- **elasticsearch-async**: Async client

#### Node.js
- **@elastic/elasticsearch**: Official Node.js client
- **bodybuilder**: Query builder

#### Java
- **elasticsearch-rest-high-level-client**: Official Java client
- **Spring Data Elasticsearch**: Spring integration

### Supporting Tools
- **Kibana**: Visualization and management
- **Logstash**: Data processing pipeline
- **Beats**: Data shippers
- **APM**: Application performance monitoring
- **Curator**: Index management

## Elasticsearch Architecture

### Index Design

**Index Structure:**
```python
from elasticsearch import Elasticsearch
from datetime import datetime

class IndexManager:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def create_product_index(self, index_name: str):
        """Create optimized product search index."""
        mapping = {
            "settings": {
                "number_of_shards": 3,
                "number_of_replicas": 2,
                "analysis": {
                    "analyzer": {
                        "autocomplete": {
                            "tokenizer": "autocomplete_tokenizer",
                            "filter": ["lowercase", "asciifolding"]
                        },
                        "autocomplete_search": {
                            "tokenizer": "standard",
                            "filter": ["lowercase", "asciifolding"]
                        }
                    },
                    "tokenizer": {
                        "autocomplete_tokenizer": {
                            "type": "edge_ngram",
                            "min_gram": 2,
                            "max_gram": 10,
                            "token_chars": ["letter", "digit"]
                        }
                    }
                }
            },
            "mappings": {
                "properties": {
                    "name": {
                        "type": "text",
                        "analyzer": "autocomplete",
                        "search_analyzer": "autocomplete_search",
                        "fields": {
                            "keyword": {"type": "keyword"},
                            "raw": {"type": "text", "analyzer": "standard"}
                        }
                    },
                    "description": {
                        "type": "text",
                        "analyzer": "standard",
                        "fields": {
                            "english": {
                                "type": "text",
                                "analyzer": "english"
                            }
                        }
                    },
                    "category": {
                        "type": "keyword"
                    },
                    "price": {
                        "type": "scaled_float",
                        "scaling_factor": 100
                    },
                    "stock": {
                        "type": "integer"
                    },
                    "rating": {
                        "type": "float"
                    },
                    "tags": {
                        "type": "keyword"
                    },
                    "location": {
                        "type": "geo_point"
                    },
                    "created_at": {
                        "type": "date"
                    },
                    "updated_at": {
                        "type": "date"
                    },
                    "is_active": {
                        "type": "boolean"
                    }
                }
            }
        }

        self.es.indices.create(index=index_name, body=mapping)
```

## Document Operations

### Indexing Documents

**Bulk Indexing:**
```python
from typing import List, Dict
from elasticsearch.helpers import bulk

class DocumentIndexer:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def index_document(self, index: str, doc_id: str, document: dict):
        """Index single document."""
        return self.es.index(
            index=index,
            id=doc_id,
            body=document,
            refresh='wait_for'  # Wait for index refresh
        )

    def bulk_index(self, index: str, documents: List[dict]):
        """Bulk index documents efficiently."""
        actions = [
            {
                "_index": index,
                "_id": doc.get('id'),
                "_source": doc
            }
            for doc in documents
        ]

        success, failed = bulk(
            self.es,
            actions,
            chunk_size=500,
            request_timeout=30
        )

        return {"success": success, "failed": failed}

    def update_document(self, index: str, doc_id: str,
                       partial_doc: dict):
        """Update document partially."""
        return self.es.update(
            index=index,
            id=doc_id,
            body={"doc": partial_doc},
            refresh='wait_for'
        )

    def delete_document(self, index: str, doc_id: str):
        """Delete document."""
        return self.es.delete(
            index=index,
            id=doc_id,
            refresh='wait_for'
        )
```

## Search Queries

### Full-Text Search

**Search Implementation:**
```python
from elasticsearch_dsl import Search, Q

class SearchEngine:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def simple_search(self, index: str, query_text: str,
                     size: int = 10):
        """Simple full-text search."""
        body = {
            "query": {
                "multi_match": {
                    "query": query_text,
                    "fields": ["name^3", "description", "tags^2"],
                    "type": "best_fields",
                    "fuzziness": "AUTO"
                }
            },
            "size": size,
            "highlight": {
                "fields": {
                    "name": {},
                    "description": {}
                }
            }
        }

        return self.es.search(index=index, body=body)

    def advanced_search(self, index: str, **kwargs):
        """Advanced search with multiple filters."""
        s = Search(using=self.es, index=index)

        # Text query
        if kwargs.get('query'):
            s = s.query(
                "multi_match",
                query=kwargs['query'],
                fields=["name^3", "description"],
                fuzziness="AUTO"
            )

        # Filters
        if kwargs.get('category'):
            s = s.filter("term", category=kwargs['category'])

        if kwargs.get('min_price') or kwargs.get('max_price'):
            price_filter = {}
            if kwargs.get('min_price'):
                price_filter['gte'] = kwargs['min_price']
            if kwargs.get('max_price'):
                price_filter['lte'] = kwargs['max_price']
            s = s.filter("range", price=price_filter)

        if kwargs.get('tags'):
            s = s.filter("terms", tags=kwargs['tags'])

        # Only active items
        s = s.filter("term", is_active=True)

        # Sorting
        if kwargs.get('sort_by'):
            s = s.sort(kwargs['sort_by'])

        # Pagination
        s = s[kwargs.get('offset', 0):kwargs.get('offset', 0) + kwargs.get('size', 10)]

        return s.execute()

    def fuzzy_search(self, index: str, query_text: str):
        """Fuzzy search for typo tolerance."""
        body = {
            "query": {
                "match": {
                    "name": {
                        "query": query_text,
                        "fuzziness": 2,
                        "prefix_length": 2,
                        "max_expansions": 50
                    }
                }
            }
        }

        return self.es.search(index=index, body=body)

    def phrase_search(self, index: str, phrase: str):
        """Search for exact phrases."""
        body = {
            "query": {
                "match_phrase": {
                    "description": {
                        "query": phrase,
                        "slop": 2  # Allow 2 words between phrase terms
                    }
                }
            }
        }

        return self.es.search(index=index, body=body)
```

### Autocomplete

**Autocomplete Implementation:**
```python
class AutocompleteEngine:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def suggest(self, index: str, prefix: str, size: int = 10):
        """Autocomplete suggestions."""
        body = {
            "query": {
                "match": {
                    "name": {
                        "query": prefix,
                        "analyzer": "autocomplete_search"
                    }
                }
            },
            "size": size,
            "_source": ["name", "category"]
        }

        response = self.es.search(index=index, body=body)

        return [
            {
                "text": hit["_source"]["name"],
                "category": hit["_source"]["category"],
                "score": hit["_score"]
            }
            for hit in response["hits"]["hits"]
        ]

    def completion_suggest(self, index: str, prefix: str):
        """Fast completion suggestions using suggest API."""
        body = {
            "suggest": {
                "product-suggest": {
                    "prefix": prefix,
                    "completion": {
                        "field": "name_suggest",
                        "size": 10,
                        "skip_duplicates": True,
                        "fuzzy": {
                            "fuzziness": 2
                        }
                    }
                }
            }
        }

        response = self.es.search(index=index, body=body)

        return [
            option["text"]
            for option in response["suggest"]["product-suggest"][0]["options"]
        ]
```

### Geospatial Search

**Location-Based Search:**
```python
class GeoSearch:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def search_nearby(self, index: str, lat: float, lon: float,
                     distance: str = "10km", size: int = 20):
        """Find documents within distance of location."""
        body = {
            "query": {
                "bool": {
                    "must": {
                        "match_all": {}
                    },
                    "filter": {
                        "geo_distance": {
                            "distance": distance,
                            "location": {
                                "lat": lat,
                                "lon": lon
                            }
                        }
                    }
                }
            },
            "sort": [
                {
                    "_geo_distance": {
                        "location": {
                            "lat": lat,
                            "lon": lon
                        },
                        "order": "asc",
                        "unit": "km"
                    }
                }
            ],
            "size": size
        }

        return self.es.search(index=index, body=body)

    def search_in_bounds(self, index: str, top_left: dict,
                        bottom_right: dict):
        """Find documents within bounding box."""
        body = {
            "query": {
                "bool": {
                    "filter": {
                        "geo_bounding_box": {
                            "location": {
                                "top_left": top_left,
                                "bottom_right": bottom_right
                            }
                        }
                    }
                }
            }
        }

        return self.es.search(index=index, body=body)
```

## Aggregations

### Faceted Search

**Aggregations for Filters:**
```python
class FacetedSearch:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def search_with_facets(self, index: str, query: str = None):
        """Search with aggregated facets."""
        body = {
            "query": {
                "multi_match": {
                    "query": query or "*",
                    "fields": ["name", "description"]
                }
            } if query else {"match_all": {}},
            "aggs": {
                "categories": {
                    "terms": {
                        "field": "category",
                        "size": 20
                    }
                },
                "price_ranges": {
                    "range": {
                        "field": "price",
                        "ranges": [
                            {"to": 50},
                            {"from": 50, "to": 100},
                            {"from": 100, "to": 200},
                            {"from": 200}
                        ]
                    }
                },
                "avg_rating": {
                    "avg": {
                        "field": "rating"
                    }
                },
                "popular_tags": {
                    "terms": {
                        "field": "tags",
                        "size": 10
                    }
                },
                "price_stats": {
                    "stats": {
                        "field": "price"
                    }
                }
            },
            "size": 20
        }

        response = self.es.search(index=index, body=body)

        return {
            "results": response["hits"]["hits"],
            "facets": {
                "categories": [
                    {
                        "name": bucket["key"],
                        "count": bucket["doc_count"]
                    }
                    for bucket in response["aggregations"]["categories"]["buckets"]
                ],
                "price_ranges": [
                    {
                        "range": f"{bucket.get('from', 0)}-{bucket.get('to', '+')}",
                        "count": bucket["doc_count"]
                    }
                    for bucket in response["aggregations"]["price_ranges"]["buckets"]
                ],
                "tags": response["aggregations"]["popular_tags"]["buckets"],
                "stats": {
                    "avg_rating": response["aggregations"]["avg_rating"]["value"],
                    "price": response["aggregations"]["price_stats"]
                }
            }
        }
```

### Analytics Aggregations

**Complex Aggregations:**
```python
class AnalyticsEngine:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def time_series_analysis(self, index: str, date_field: str = "created_at"):
        """Time series analysis with date histogram."""
        body = {
            "aggs": {
                "sales_over_time": {
                    "date_histogram": {
                        "field": date_field,
                        "calendar_interval": "day",
                        "format": "yyyy-MM-dd"
                    },
                    "aggs": {
                        "revenue": {
                            "sum": {"field": "price"}
                        },
                        "avg_rating": {
                            "avg": {"field": "rating"}
                        }
                    }
                }
            },
            "size": 0
        }

        return self.es.search(index=index, body=body)

    def top_sellers(self, index: str, size: int = 10):
        """Find top selling products."""
        body = {
            "aggs": {
                "top_products": {
                    "terms": {
                        "field": "name.keyword",
                        "size": size,
                        "order": {"total_revenue": "desc"}
                    },
                    "aggs": {
                        "total_revenue": {
                            "sum": {"field": "price"}
                        }
                    }
                }
            },
            "size": 0
        }

        return self.es.search(index=index, body=body)
```

## Relevance Tuning

### Boosting and Scoring

**Custom Scoring:**
```python
class RelevanceTuner:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def boosted_search(self, index: str, query: str):
        """Search with field boosting and function scoring."""
        body = {
            "query": {
                "function_score": {
                    "query": {
                        "multi_match": {
                            "query": query,
                            "fields": [
                                "name^5",          # Boost name matches
                                "description^2",   # Moderate boost for description
                                "tags^3"           # High boost for tag matches
                            ],
                            "type": "best_fields",
                            "fuzziness": "AUTO"
                        }
                    },
                    "functions": [
                        {
                            # Boost popular items
                            "field_value_factor": {
                                "field": "rating",
                                "factor": 1.2,
                                "modifier": "sqrt",
                                "missing": 1
                            }
                        },
                        {
                            # Boost recent items
                            "gauss": {
                                "created_at": {
                                    "origin": "now",
                                    "scale": "30d",
                                    "decay": 0.5
                                }
                            }
                        },
                        {
                            # Boost in-stock items
                            "filter": {"range": {"stock": {"gt": 0}}},
                            "weight": 1.5
                        }
                    ],
                    "score_mode": "multiply",
                    "boost_mode": "multiply"
                }
            }
        }

        return self.es.search(index=index, body=body)

    def personalized_search(self, index: str, query: str,
                          user_preferences: dict):
        """Personalized search based on user preferences."""
        must = [
            {
                "multi_match": {
                    "query": query,
                    "fields": ["name", "description"]
                }
            }
        ]

        should = []

        # Boost preferred categories
        if user_preferences.get('favorite_categories'):
            should.append({
                "terms": {
                    "category": user_preferences['favorite_categories'],
                    "boost": 2.0
                }
            })

        # Boost price range
        if user_preferences.get('price_range'):
            should.append({
                "range": {
                    "price": {
                        **user_preferences['price_range'],
                        "boost": 1.5
                    }
                }
            })

        body = {
            "query": {
                "bool": {
                    "must": must,
                    "should": should
                }
            }
        }

        return self.es.search(index=index, body=body)
```

## Index Management

### Index Lifecycle

**Index Operations:**
```python
class IndexLifecycle:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def reindex(self, source_index: str, dest_index: str):
        """Reindex data to new index."""
        body = {
            "source": {"index": source_index},
            "dest": {"index": dest_index}
        }

        return self.es.reindex(body=body, wait_for_completion=False)

    def create_alias(self, index: str, alias: str):
        """Create index alias."""
        return self.es.indices.put_alias(
            index=index,
            name=alias
        )

    def zero_downtime_reindex(self, old_index: str, new_index: str,
                             alias: str):
        """Reindex with zero downtime using aliases."""
        # 1. Create new index
        # 2. Reindex from old to new
        task = self.reindex(old_index, new_index)

        # 3. Wait for completion (or monitor task)
        # 4. Atomic alias swap
        body = {
            "actions": [
                {"remove": {"index": old_index, "alias": alias}},
                {"add": {"index": new_index, "alias": alias}}
            ]
        }

        return self.es.indices.update_aliases(body=body)

    def optimize_index(self, index: str):
        """Optimize index for search performance."""
        return self.es.indices.forcemerge(
            index=index,
            max_num_segments=1
        )

    def close_index(self, index: str):
        """Close index to save resources."""
        return self.es.indices.close(index=index)

    def delete_old_indices(self, pattern: str, days: int = 30):
        """Delete indices older than specified days."""
        from datetime import datetime, timedelta

        cutoff_date = datetime.now() - timedelta(days=days)

        indices = self.es.indices.get(index=pattern)
        for index_name in indices:
            index_date = self._extract_date_from_name(index_name)
            if index_date and index_date < cutoff_date:
                self.es.indices.delete(index=index_name)
```

## Performance Optimization

### Query Optimization

**Optimization Techniques:**
```python
class QueryOptimizer:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def use_filters_over_queries(self, index: str):
        """Use filters for exact matches (cached)."""
        body = {
            "query": {
                "bool": {
                    "must": [
                        {"match": {"description": "laptop"}}
                    ],
                    "filter": [  # Filters are cached
                        {"term": {"category": "electronics"}},
                        {"range": {"price": {"lte": 1000}}}
                    ]
                }
            }
        }

        return self.es.search(index=index, body=body)

    def limit_source_fields(self, index: str, query: str):
        """Return only needed fields."""
        body = {
            "query": {"match": {"name": query}},
            "_source": ["name", "price", "category"]  # Only return these
        }

        return self.es.search(index=index, body=body)

    def use_scroll_for_large_results(self, index: str, query: dict):
        """Use scroll API for large result sets."""
        page = self.es.search(
            index=index,
            body=query,
            scroll='2m',
            size=1000
        )

        sid = page['_scroll_id']
        scroll_size = len(page['hits']['hits'])

        results = []

        while scroll_size > 0:
            results.extend(page['hits']['hits'])

            page = self.es.scroll(scroll_id=sid, scroll='2m')
            sid = page['_scroll_id']
            scroll_size = len(page['hits']['hits'])

        # Clear scroll
        self.es.clear_scroll(scroll_id=sid)

        return results
```

## Monitoring and Debugging

### Cluster Health

**Monitoring Tools:**
```python
class ClusterMonitor:
    def __init__(self, es_client: Elasticsearch):
        self.es = es_client

    def get_cluster_health(self) -> dict:
        """Get cluster health status."""
        health = self.es.cluster.health()

        return {
            "status": health["status"],
            "nodes": health["number_of_nodes"],
            "active_shards": health["active_shards"],
            "relocating_shards": health["relocating_shards"],
            "unassigned_shards": health["unassigned_shards"]
        }

    def get_index_stats(self, index: str) -> dict:
        """Get index statistics."""
        stats = self.es.indices.stats(index=index)

        return {
            "size": stats["_all"]["total"]["store"]["size_in_bytes"],
            "documents": stats["_all"]["total"]["docs"]["count"],
            "deleted_docs": stats["_all"]["total"]["docs"]["deleted"]
        }

    def analyze_query(self, index: str, query: dict):
        """Explain query execution."""
        return self.es.indices.validate_query(
            index=index,
            body=query,
            explain=True
        )

    def get_slow_queries(self, index: str):
        """Check slow query logs."""
        settings = self.es.indices.get_settings(index=index)
        return settings[index]["settings"]["index"].get("search", {}).get("slowlog", {})
```

## Best Practices

### 1. Index Design
- Choose appropriate data types
- Use keyword for exact matches, text for full-text
- Design for your query patterns
- Use index templates for consistent settings
- Implement proper shard allocation

### 2. Query Performance
- Use filters for exact matches (cached)
- Limit returned fields with _source filtering
- Use pagination with from/size or search_after
- Avoid deep pagination
- Use scroll API for large exports

### 3. Indexing Performance
- Use bulk API for batch indexing
- Adjust refresh_interval for write-heavy loads
- Disable replicas during bulk indexing
- Use async indexing when possible
- Optimize number of shards

### 4. Relevance Tuning
- Test different analyzers
- Use field boosting appropriately
- Implement function scoring for business rules
- Monitor click-through rates
- A/B test relevance changes

### 5. Security
- Enable authentication and authorization
- Use TLS/SSL encryption
- Implement field-level security
- Audit access logs
- Regular security updates

### 6. Monitoring
- Track cluster health
- Monitor query performance
- Set up alerting for issues
- Track slow queries
- Monitor resource usage

### 7. High Availability
- Use replicas for redundancy
- Implement cross-cluster replication
- Configure snapshot backups
- Use dedicated master nodes
- Implement proper shard allocation

## Common Anti-Patterns to Avoid

1. **Over-sharding**: Too many shards decreases performance
2. **Deep pagination**: Use search_after instead of from/size
3. **Wildcard prefix queries**: Very expensive, use edge ngrams
4. **Large documents**: Split into smaller documents
5. **Not using filters**: Filters are cached, queries are not
6. **Ignoring cluster health**: Monitor and fix yellow/red status
7. **No backup strategy**: Always implement snapshots

## Deliverables

When implementing search, ensure you deliver:

1. **Index Configuration**
   - Mappings and settings
   - Analyzers and tokenizers
   - Shard and replica configuration

2. **Search Implementation**
   - Query builders and DSL
   - Autocomplete functionality
   - Faceted search
   - Relevance tuning

3. **Integration Code**
   - Application integration
   - Error handling
   - Retry logic
   - Connection pooling

4. **Monitoring Setup**
   - Cluster health monitoring
   - Query performance tracking
   - Alerting configuration

5. **Documentation**
   - Index schema documentation
   - Query examples
   - Troubleshooting guide
   - Performance tuning guide

## Quality Checklist

Before completing a search implementation, verify:

- [ ] Index mappings optimized for query patterns
- [ ] Analyzers configured for language/use case
- [ ] Sharding strategy appropriate for data size
- [ ] Replication configured for high availability
- [ ] Queries use filters where appropriate
- [ ] Pagination implemented correctly
- [ ] Autocomplete functionality tested
- [ ] Faceted search working correctly
- [ ] Relevance scoring tuned and tested
- [ ] Security (authentication, TLS) enabled
- [ ] Monitoring and alerting configured
- [ ] Backup snapshots scheduled
- [ ] Performance tested under load
- [ ] Documentation complete and accurate
- [ ] Error handling comprehensive
