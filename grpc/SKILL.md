---
name: grpc
description: "Build high-performance gRPC services with Protocol Buffers, streaming, interceptors, and error handling. Implement unary, server streaming, client streaming, and bidirectional streaming."
---

# gRPC Skill

Build efficient, type-safe RPC services with gRPC and Protocol Buffers.

## When to Use

Use this skill when the user wants to:
- Build high-performance microservices
- Implement RPC (Remote Procedure Call) services
- Create streaming APIs (server, client, bidirectional)
- Build polyglot systems (multi-language services)
- Implement efficient binary protocols
- Build low-latency real-time services
- Create strongly-typed service contracts
- Implement service-to-service communication
- Build distributed systems with strict contracts

## gRPC Overview

### Advantages
- **Performance**: Binary protocol (Protocol Buffers)
- **Streaming**: Built-in support for streaming
- **Type Safety**: Strongly-typed contracts
- **Code Generation**: Auto-generate client/server code
- **Multi-Language**: Support for many languages
- **HTTP/2**: Multiplexing, header compression

### Communication Patterns
1. **Unary RPC**: Single request, single response
2. **Server Streaming**: Single request, stream of responses
3. **Client Streaming**: Stream of requests, single response
4. **Bidirectional Streaming**: Stream of requests and responses

## Protocol Buffers

### Define Service (.proto file)
```protobuf
syntax = "proto3";

package user;

// User service definition
service UserService {
  // Unary RPC
  rpc GetUser (GetUserRequest) returns (User);
  rpc CreateUser (CreateUserRequest) returns (User);
  rpc UpdateUser (UpdateUserRequest) returns (User);
  rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);

  // Server streaming
  rpc ListUsers (ListUsersRequest) returns (stream User);

  // Client streaming
  rpc CreateUsers (stream CreateUserRequest) returns (CreateUsersResponse);

  // Bidirectional streaming
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}

// Message definitions
message User {
  string id = 1;
  string username = 2;
  string email = 3;
  int64 created_at = 4;
  repeated string roles = 5;
}

message GetUserRequest {
  string id = 1;
}

message CreateUserRequest {
  string username = 1;
  string email = 2;
  string password = 3;
}

message UpdateUserRequest {
  string id = 1;
  optional string username = 2;
  optional string email = 3;
}

message DeleteUserRequest {
  string id = 1;
}

message DeleteUserResponse {
  bool success = 1;
}

message ListUsersRequest {
  int32 page_size = 1;
  string page_token = 2;
}

message CreateUsersResponse {
  repeated User users = 1;
  int32 count = 2;
}

message ChatMessage {
  string user_id = 1;
  string text = 2;
  int64 timestamp = 3;
}
```

## Server Implementation

### Python (grpcio)
```python
import grpc
from concurrent import futures
import user_pb2
import user_pb2_grpc
import time

class UserServiceServicer(user_pb2_grpc.UserServiceServicer):
    def __init__(self):
        self.users = {}

    def GetUser(self, request, context):
        """Unary RPC: Get single user."""
        user_id = request.id

        if user_id not in self.users:
            context.set_code(grpc.StatusCode.NOT_FOUND)
            context.set_details(f'User {user_id} not found')
            return user_pb2.User()

        return self.users[user_id]

    def CreateUser(self, request, context):
        """Unary RPC: Create user."""
        user = user_pb2.User(
            id=str(uuid.uuid4()),
            username=request.username,
            email=request.email,
            created_at=int(time.time())
        )

        self.users[user.id] = user
        return user

    def ListUsers(self, request, context):
        """Server streaming: Stream all users."""
        for user in self.users.values():
            yield user
            time.sleep(0.1)  # Simulate delay

    def CreateUsers(self, request_iterator, context):
        """Client streaming: Receive stream of users."""
        created_users = []

        for user_request in request_iterator:
            user = user_pb2.User(
                id=str(uuid.uuid4()),
                username=user_request.username,
                email=user_request.email,
                created_at=int(time.time())
            )
            self.users[user.id] = user
            created_users.append(user)

        return user_pb2.CreateUsersResponse(
            users=created_users,
            count=len(created_users)
        )

    def Chat(self, request_iterator, context):
        """Bidirectional streaming: Chat."""
        for message in request_iterator:
            # Echo message back (in real app, broadcast to others)
            response = user_pb2.ChatMessage(
                user_id=message.user_id,
                text=f"Echo: {message.text}",
                timestamp=int(time.time())
            )
            yield response

def serve():
    server = grpc.server(futures.ThreadPoolExecutor(max_workers=10))
    user_pb2_grpc.add_UserServiceServicer_to_server(
        UserServiceServicer(), server
    )

    server.add_insecure_port('[::]:50051')
    print('Server started on port 50051')
    server.start()
    server.wait_for_termination()

if __name__ == '__main__':
    serve()
```

### Go Implementation
```go
package main

import (
    "context"
    "io"
    "log"
    "net"
    "time"

    "google.golang.org/grpc"
    pb "path/to/user"
)

type server struct {
    pb.UnimplementedUserServiceServer
    users map[string]*pb.User
}

// Unary RPC
func (s *server) GetUser(ctx context.Context, req *pb.GetUserRequest) (*pb.User, error) {
    user, ok := s.users[req.Id]
    if !ok {
        return nil, status.Errorf(codes.NotFound, "User not found")
    }
    return user, nil
}

func (s *server) CreateUser(ctx context.Context, req *pb.CreateUserRequest) (*pb.User, error) {
    user := &pb.User{
        Id:        uuid.New().String(),
        Username:  req.Username,
        Email:     req.Email,
        CreatedAt: time.Now().Unix(),
    }
    s.users[user.Id] = user
    return user, nil
}

// Server streaming
func (s *server) ListUsers(req *pb.ListUsersRequest, stream pb.UserService_ListUsersServer) error {
    for _, user := range s.users {
        if err := stream.Send(user); err != nil {
            return err
        }
        time.Sleep(100 * time.Millisecond)
    }
    return nil
}

// Client streaming
func (s *server) CreateUsers(stream pb.UserService_CreateUsersServer) error {
    var users []*pb.User

    for {
        req, err := stream.Recv()
        if err == io.EOF {
            return stream.SendAndClose(&pb.CreateUsersResponse{
                Users: users,
                Count: int32(len(users)),
            })
        }
        if err != nil {
            return err
        }

        user := &pb.User{
            Id:        uuid.New().String(),
            Username:  req.Username,
            Email:     req.Email,
            CreatedAt: time.Now().Unix(),
        }
        s.users[user.Id] = user
        users = append(users, user)
    }
}

// Bidirectional streaming
func (s *server) Chat(stream pb.UserService_ChatServer) error {
    for {
        msg, err := stream.Recv()
        if err == io.EOF {
            return nil
        }
        if err != nil {
            return err
        }

        response := &pb.ChatMessage{
            UserId:    msg.UserId,
            Text:      "Echo: " + msg.Text,
            Timestamp: time.Now().Unix(),
        }

        if err := stream.Send(response); err != nil {
            return err
        }
    }
}

func main() {
    lis, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("Failed to listen: %v", err)
    }

    s := grpc.NewServer()
    pb.RegisterUserServiceServer(s, &server{
        users: make(map[string]*pb.User),
    })

    log.Println("Server started on :50051")
    if err := s.Serve(lis); err != nil {
        log.Fatalf("Failed to serve: %v", err)
    }
}
```

## Client Implementation

### Python Client
```python
import grpc
import user_pb2
import user_pb2_grpc

def run():
    channel = grpc.insecure_channel('localhost:50051')
    stub = user_pb2_grpc.UserServiceStub(channel)

    # Unary call
    response = stub.GetUser(user_pb2.GetUserRequest(id='123'))
    print(f'User: {response.username}')

    # Server streaming
    for user in stub.ListUsers(user_pb2.ListUsersRequest()):
        print(f'User: {user.username}')

    # Client streaming
    def generate_users():
        for i in range(5):
            yield user_pb2.CreateUserRequest(
                username=f'user{i}',
                email=f'user{i}@example.com',
                password='password'
            )

    response = stub.CreateUsers(generate_users())
    print(f'Created {response.count} users')

    # Bidirectional streaming
    def generate_messages():
        for i in range(5):
            yield user_pb2.ChatMessage(
                user_id='user1',
                text=f'Message {i}',
                timestamp=int(time.time())
            )

    responses = stub.Chat(generate_messages())
    for response in responses:
        print(f'Received: {response.text}')

if __name__ == '__main__':
    run()
```

### Node.js Client
```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

const packageDefinition = protoLoader.loadSync('user.proto');
const userProto = grpc.loadPackageDefinition(packageDefinition).user;

const client = new userProto.UserService(
    'localhost:50051',
    grpc.credentials.createInsecure()
);

// Unary call
client.getUser({ id: '123' }, (err, response) => {
    if (err) {
        console.error('Error:', err);
        return;
    }
    console.log('User:', response);
});

// Server streaming
const call = client.listUsers({});
call.on('data', (user) => {
    console.log('User:', user);
});
call.on('end', () => {
    console.log('Stream ended');
});

// Client streaming
const createCall = client.createUsers((err, response) => {
    console.log(`Created ${response.count} users`);
});

for (let i = 0; i < 5; i++) {
    createCall.write({
        username: `user${i}`,
        email: `user${i}@example.com`,
        password: 'password'
    });
}
createCall.end();

// Bidirectional streaming
const chatCall = client.chat();

chatCall.on('data', (message) => {
    console.log('Received:', message.text);
});

for (let i = 0; i < 5; i++) {
    chatCall.write({
        user_id: 'user1',
        text: `Message ${i}`,
        timestamp: Date.now()
    });
}
chatCall.end();
```

## Interceptors (Middleware)

### Server Interceptor (Python)
```python
import grpc

class AuthInterceptor(grpc.ServerInterceptor):
    def intercept_service(self, continuation, handler_call_details):
        # Get metadata
        metadata = dict(handler_call_details.invocation_metadata)

        # Check authorization
        token = metadata.get('authorization')
        if not token or not verify_token(token):
            context = grpc.ServicerContext()
            context.abort(grpc.StatusCode.UNAUTHENTICATED, 'Invalid token')

        return continuation(handler_call_details)

# Add interceptor to server
server = grpc.server(
    futures.ThreadPoolExecutor(max_workers=10),
    interceptors=[AuthInterceptor()]
)
```

### Client Interceptor (Python)
```python
class ClientInterceptor(grpc.UnaryUnaryClientInterceptor):
    def intercept_unary_unary(self, continuation, client_call_details, request):
        # Add metadata
        metadata = []
        if client_call_details.metadata is not None:
            metadata = list(client_call_details.metadata)

        metadata.append(('authorization', f'Bearer {get_token()}'))

        client_call_details = client_call_details._replace(metadata=metadata)

        return continuation(client_call_details, request)

# Use interceptor
channel = grpc.insecure_channel('localhost:50051')
channel = grpc.intercept_channel(channel, ClientInterceptor())
stub = user_pb2_grpc.UserServiceStub(channel)
```

## Error Handling

### Custom Error Codes
```python
def GetUser(self, request, context):
    user_id = request.id

    if not user_id:
        context.set_code(grpc.StatusCode.INVALID_ARGUMENT)
        context.set_details('User ID is required')
        return user_pb2.User()

    if user_id not in self.users:
        context.set_code(grpc.StatusCode.NOT_FOUND)
        context.set_details(f'User {user_id} not found')
        return user_pb2.User()

    return self.users[user_id]

# Client error handling
try:
    response = stub.GetUser(user_pb2.GetUserRequest(id='123'))
except grpc.RpcError as e:
    print(f'Error: {e.code()}, {e.details()}')
```

## Authentication & Security

### TLS/SSL
```python
# Server with TLS
server_credentials = grpc.ssl_server_credentials(
    [(private_key, certificate_chain)]
)
server.add_secure_port('[::]:50051', server_credentials)

# Client with TLS
channel_credentials = grpc.ssl_channel_credentials(
    root_certificates=ca_cert
)
channel = grpc.secure_channel('localhost:50051', channel_credentials)
```

### JWT Authentication
```python
class JWTAuthInterceptor(grpc.ServerInterceptor):
    def intercept_service(self, continuation, handler_call_details):
        metadata = dict(handler_call_details.invocation_metadata)
        token = metadata.get('authorization', '').replace('Bearer ', '')

        try:
            payload = jwt.decode(token, SECRET_KEY, algorithms=['HS256'])
            # Add user to context
            return continuation(handler_call_details)
        except jwt.InvalidTokenError:
            context = grpc.ServicerContext()
            context.abort(grpc.StatusCode.UNAUTHENTICATED, 'Invalid token')
```

## Load Balancing

### Client-Side Load Balancing
```python
# Python client with load balancing
channel = grpc.insecure_channel(
    'dns:///my-service:50051',
    options=[
        ('grpc.lb_policy_name', 'round_robin'),
    ]
)
```

```javascript
// Node.js client with load balancing
const client = new userProto.UserService(
    'dns:///my-service:50051',
    grpc.credentials.createInsecure(),
    {
        'grpc.lb_policy_name': 'round_robin'
    }
);
```

## Testing

### Unit Testing (Python)
```python
import unittest
from unittest.mock import Mock
import grpc_testing

class TestUserService(unittest.TestCase):
    def setUp(self):
        self.service = UserServiceServicer()
        self.server = grpc_testing.server_from_dictionary(
            {user_pb2.DESCRIPTOR.services_by_name['UserService']: self.service},
            grpc_testing.strict_real_time()
        )

    def test_get_user(self):
        request = user_pb2.GetUserRequest(id='123')
        method = self.server.invoke_unary_unary(
            user_pb2.DESCRIPTOR.services_by_name['UserService'].methods_by_name['GetUser'],
            {},
            request,
            None
        )

        response, metadata, code, details = method.termination()
        self.assertEqual(code, grpc.StatusCode.OK)
```

## Best Practices

### Protocol Buffers
- Use **semantic versioning** for .proto files
- Mark fields as **optional** when appropriate
- Use **reserved** for deprecated fields
- Use **enums** for fixed sets of values
- Create **well-structured message hierarchies**

### Performance
- Use **streaming** for large datasets
- Implement **connection pooling**
- Enable **compression** when beneficial
- Use **binary protocol** (default)
- Implement **caching** where appropriate

### Security
- Always use **TLS/SSL** in production
- Implement **authentication** (JWT, mTLS)
- Validate **all inputs**
- Implement **rate limiting**
- Use **interceptors** for cross-cutting concerns

### Error Handling
- Use appropriate **status codes**
- Provide **meaningful error messages**
- Implement **retry logic** with backoff
- Handle **timeouts** gracefully
- Log errors for debugging

## Monitoring

### Metrics to Track
- Request count and rate
- Response latency
- Error rate
- Active connections
- Stream duration

### OpenTelemetry Integration
```python
from opentelemetry import trace
from opentelemetry.instrumentation.grpc import GrpcInstrumentorServer

# Instrument gRPC server
GrpcInstrumentorServer().instrument()

# Trace will be automatic
```

## Common Use Cases

1. **Microservices**: Service-to-service communication
2. **Real-Time Systems**: Low-latency streaming
3. **Mobile Apps**: Efficient binary protocol
4. **IoT**: Lightweight communication
5. **Distributed Systems**: Type-safe contracts
6. **Polyglot Systems**: Multi-language support

## Deliverables

- .proto service definitions
- Generated client/server code
- Service implementation
- Client implementation
- Interceptors for auth/logging
- Error handling
- TLS configuration
- Testing suite
- Documentation

## Quality Checklist

- .proto files are well-designed
- TLS/SSL is configured for production
- Authentication is implemented
- Error handling is comprehensive
- Streaming is used appropriately
- Interceptors handle cross-cutting concerns
- Load balancing is configured
- Testing coverage is adequate
- Monitoring is in place
- Documentation is complete
