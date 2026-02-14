---
name: graphql
description: "Build GraphQL APIs with schema design, resolvers, queries, mutations, subscriptions. Implement DataLoader, authentication, and optimize N+1 queries."
---

# GraphQL API Skill

Design and implement efficient, type-safe GraphQL APIs.

## When to Use

Use this skill when the user wants to:
- Build GraphQL APIs
- Design GraphQL schemas
- Implement queries, mutations, and subscriptions
- Optimize GraphQL performance (N+1 problem)
- Implement GraphQL authentication and authorization
- Build real-time GraphQL subscriptions
- Integrate GraphQL with databases
- Implement pagination and filtering
- Use DataLoader for batching
- Build GraphQL federation

## GraphQL Basics

### Schema Definition
```graphql
type User {
  id: ID!
  username: String!
  email: String!
  posts: [Post!]!
  createdAt: DateTime!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  published: Boolean!
  createdAt: DateTime!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
  createdAt: DateTime!
}

type Query {
  user(id: ID!): User
  users(limit: Int, offset: Int): [User!]!
  post(id: ID!): Post
  posts(published: Boolean, limit: Int): [Post!]!
}

type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  createPost(input: CreatePostInput!): Post!
  publishPost(id: ID!): Post!
}

type Subscription {
  postPublished: Post!
  commentAdded(postId: ID!): Comment!
}

input CreateUserInput {
  username: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  username: String
  email: String
}

input CreatePostInput {
  title: String!
  content: String!
  published: Boolean = false
}
```

## Server Implementation (Node.js)

### Apollo Server Setup
```javascript
const { ApolloServer } = require('apollo-server');
const { makeExecutableSchema } = require('@graphql-tools/schema');

const typeDefs = `
  # Schema definitions here
`;

const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      return await context.db.user.findUnique({ where: { id } });
    },

    users: async (parent, { limit = 10, offset = 0 }, context) => {
      return await context.db.user.findMany({
        take: limit,
        skip: offset
      });
    },

    post: async (parent, { id }, context) => {
      return await context.db.post.findUnique({ where: { id } });
    }
  },

  Mutation: {
    createUser: async (parent, { input }, context) => {
      // Check authentication
      if (!context.user) {
        throw new Error('Not authenticated');
      }

      // Hash password
      const hashedPassword = await hashPassword(input.password);

      return await context.db.user.create({
        data: {
          ...input,
          password: hashedPassword
        }
      });
    },

    createPost: async (parent, { input }, context) => {
      if (!context.user) {
        throw new Error('Not authenticated');
      }

      return await context.db.post.create({
        data: {
          ...input,
          authorId: context.user.id
        }
      });
    }
  },

  User: {
    posts: async (parent, args, context) => {
      // Use DataLoader to avoid N+1 queries
      return await context.loaders.postsByUserId.load(parent.id);
    }
  },

  Post: {
    author: async (parent, args, context) => {
      return await context.loaders.userById.load(parent.authorId);
    },

    comments: async (parent, args, context) => {
      return await context.loaders.commentsByPostId.load(parent.id);
    }
  }
};

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => ({
    user: getUserFromToken(req.headers.authorization),
    db: prisma,
    loaders: createLoaders()
  })
});

server.listen().then(({ url }) => {
  console.log(`Server ready at ${url}`);
});
```

## Python Implementation (Strawberry)

```python
import strawberry
from typing import List, Optional
from strawberry.types import Info

@strawberry.type
class User:
    id: strawberry.ID
    username: str
    email: str
    created_at: str

    @strawberry.field
    async def posts(self, info: Info) -> List['Post']:
        # Use DataLoader
        return await info.context['loaders']['posts_by_user_id'].load(self.id)

@strawberry.type
class Post:
    id: strawberry.ID
    title: str
    content: str
    published: bool
    created_at: str

    @strawberry.field
    async def author(self, info: Info) -> User:
        return await info.context['loaders']['user_by_id'].load(self.author_id)

@strawberry.type
class Query:
    @strawberry.field
    async def user(self, id: strawberry.ID, info: Info) -> Optional[User]:
        return await info.context['db'].get_user(id)

    @strawberry.field
    async def users(self, limit: int = 10, info: Info) -> List[User]:
        return await info.context['db'].get_users(limit=limit)

@strawberry.type
class Mutation:
    @strawberry.mutation
    async def create_user(
        self,
        username: str,
        email: str,
        password: str,
        info: Info
    ) -> User:
        # Hash password
        hashed_password = hash_password(password)

        # Create user
        user = await info.context['db'].create_user(
            username=username,
            email=email,
            password=hashed_password
        )

        return user

# Create schema
schema = strawberry.Schema(query=Query, mutation=Mutation)
```

## DataLoader (Solving N+1 Problem)

### JavaScript Implementation
```javascript
const DataLoader = require('dataloader');

function createLoaders(db) {
  return {
    userById: new DataLoader(async (userIds) => {
      const users = await db.user.findMany({
        where: { id: { in: userIds } }
      });

      // Return in same order as requested
      const userMap = new Map(users.map(u => [u.id, u]));
      return userIds.map(id => userMap.get(id));
    }),

    postsByUserId: new DataLoader(async (userIds) => {
      const posts = await db.post.findMany({
        where: { authorId: { in: userIds } }
      });

      // Group by userId
      const postsByUser = new Map();
      for (const post of posts) {
        if (!postsByUser.has(post.authorId)) {
          postsByUser.set(post.authorId, []);
        }
        postsByUser.get(post.authorId).push(post);
      }

      return userIds.map(id => postsByUser.get(id) || []);
    }),

    commentsByPostId: new DataLoader(async (postIds) => {
      const comments = await db.comment.findMany({
        where: { postId: { in: postIds } }
      });

      const commentsByPost = new Map();
      for (const comment of comments) {
        if (!commentsByPost.has(comment.postId)) {
          commentsByPost.set(comment.postId, []);
        }
        commentsByPost.get(comment.postId).push(comment);
      }

      return postIds.map(id => commentsByPost.get(id) || []);
    })
  };
}
```

## Authentication & Authorization

### Context-Based Auth
```javascript
const { ApolloServer } = require('apollo-server');
const jwt = require('jsonwebtoken');

function getUserFromToken(token) {
  if (!token) return null;

  try {
    const decoded = jwt.verify(token.replace('Bearer ', ''), SECRET_KEY);
    return decoded;
  } catch (err) {
    return null;
  }
}

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const user = getUserFromToken(req.headers.authorization);
    return { user, db: prisma, loaders: createLoaders() };
  }
});
```

### Directive-Based Auth
```graphql
directive @auth(requires: Role = USER) on FIELD_DEFINITION

enum Role {
  ADMIN
  USER
  GUEST
}

type Query {
  publicData: String
  userData: String @auth(requires: USER)
  adminData: String @auth(requires: ADMIN)
}
```

```javascript
const { SchemaDirectiveVisitor } = require('graphql-tools');

class AuthDirective extends SchemaDirectiveVisitor {
  visitFieldDefinition(field) {
    const { resolve = defaultFieldResolver } = field;
    const { requires } = this.args;

    field.resolve = function(...args) {
      const context = args[2];

      if (!context.user) {
        throw new Error('Not authenticated');
      }

      if (requires && context.user.role !== requires) {
        throw new Error('Not authorized');
      }

      return resolve.apply(this, args);
    };
  }
}
```

## Subscriptions (Real-Time)

### Server Setup
```javascript
const { createServer } = require('http');
const { ApolloServer } = require('apollo-server-express');
const { makeExecutableSchema } = require('@graphql-tools/schema');
const { WebSocketServer } = require('ws');
const { useServer } = require('graphql-ws/lib/use/ws');
const { PubSub } = require('graphql-subscriptions');

const pubsub = new PubSub();

const typeDefs = `
  type Subscription {
    postPublished: Post!
    messageAdded(roomId: ID!): Message!
  }
`;

const resolvers = {
  Subscription: {
    postPublished: {
      subscribe: () => pubsub.asyncIterator(['POST_PUBLISHED'])
    },

    messageAdded: {
      subscribe: (parent, { roomId }) => {
        return pubsub.asyncIterator([`MESSAGE_ADDED_${roomId}`]);
      }
    }
  },

  Mutation: {
    publishPost: async (parent, { id }, context) => {
      const post = await context.db.post.update({
        where: { id },
        data: { published: true }
      });

      // Trigger subscription
      pubsub.publish('POST_PUBLISHED', { postPublished: post });

      return post;
    }
  }
};

const schema = makeExecutableSchema({ typeDefs, resolvers });

const server = new ApolloServer({ schema });
const httpServer = createServer();

server.start().then(() => {
  server.applyMiddleware({ app: httpServer });

  const wsServer = new WebSocketServer({
    server: httpServer,
    path: '/graphql'
  });

  useServer({ schema }, wsServer);

  httpServer.listen(4000);
});
```

### Client (React)
```javascript
import { useSubscription, gql } from '@apollo/client';

const POST_PUBLISHED_SUBSCRIPTION = gql`
  subscription OnPostPublished {
    postPublished {
      id
      title
      author {
        username
      }
    }
  }
`;

function PostFeed() {
  const { data, loading } = useSubscription(POST_PUBLISHED_SUBSCRIPTION);

  if (loading) return <div>Loading...</div>;

  return (
    <div>
      <h3>New Post: {data.postPublished.title}</h3>
      <p>By: {data.postPublished.author.username}</p>
    </div>
  );
}
```

## Pagination

### Cursor-Based Pagination
```graphql
type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type PostEdge {
  node: Post!
  cursor: String!
}

type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type Query {
  posts(first: Int, after: String, last: Int, before: String): PostConnection!
}
```

```javascript
const resolvers = {
  Query: {
    posts: async (parent, { first, after, last, before }, context) => {
      const limit = first || last || 10;
      const cursor = after || before;

      const posts = await context.db.post.findMany({
        take: limit + 1,
        skip: cursor ? 1 : 0,
        cursor: cursor ? { id: cursor } : undefined,
        orderBy: { createdAt: 'desc' }
      });

      const hasNextPage = posts.length > limit;
      const edges = posts.slice(0, limit).map(post => ({
        node: post,
        cursor: post.id
      }));

      return {
        edges,
        pageInfo: {
          hasNextPage,
          hasPreviousPage: !!cursor,
          startCursor: edges[0]?.cursor,
          endCursor: edges[edges.length - 1]?.cursor
        },
        totalCount: await context.db.post.count()
      };
    }
  }
};
```

## Best Practices

### Schema Design
- Use **clear naming** conventions
- Define **nullable vs non-nullable** fields carefully
- Use **Input types** for mutations
- Create **reusable fragments**
- Version your schema when needed

### Performance
- Use **DataLoader** to batch database queries
- Implement **query complexity** limits
- Add **query depth** limits
- Use **persisted queries**
- Implement **caching** (Redis)

### Security
- **Validate inputs** in resolvers
- Implement **rate limiting**
- Use **query cost analysis**
- Enable **CORS** properly
- **Sanitize** error messages

### Error Handling
```javascript
const { ApolloError } = require('apollo-server');

class NotFoundError extends ApolloError {
  constructor(message) {
    super(message, 'NOT_FOUND');
  }
}

const resolvers = {
  Query: {
    user: async (parent, { id }, context) => {
      const user = await context.db.user.findUnique({ where: { id } });

      if (!user) {
        throw new NotFoundError(`User with id ${id} not found`);
      }

      return user;
    }
  }
};
```

## Testing

### Unit Testing
```javascript
const { createTestClient } = require('apollo-server-testing');

describe('GraphQL Resolvers', () => {
  it('fetches user by id', async () => {
    const { query } = createTestClient(server);

    const GET_USER = gql`
      query GetUser($id: ID!) {
        user(id: $id) {
          id
          username
        }
      }
    `;

    const { data } = await query({
      query: GET_USER,
      variables: { id: '1' }
    });

    expect(data.user).toEqual({
      id: '1',
      username: 'testuser'
    });
  });
});
```

## Deliverables

- GraphQL schema definition
- Resolver implementations
- DataLoader setup
- Authentication/authorization
- Subscription handlers (if needed)
- Error handling
- Testing suite
- Documentation

## Quality Checklist

- Schema is well-designed and documented
- N+1 queries are solved with DataLoader
- Authentication is implemented
- Input validation is in place
- Error handling is comprehensive
- Query complexity limits configured
- Subscriptions work correctly (if used)
- Testing coverage is adequate
- Performance optimizations applied
