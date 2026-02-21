# gRPC vs REST

> **References:** [gRPC Documentation](https://grpc.io/docs/) | [Protocol Buffers](https://protobuf.dev/) | [Uber gRPC Migration](https://www.uber.com/en-US/blog/distributed-systems-design-with-grpc/)

---

## What Is gRPC?

gRPC (Google Remote Procedure Call) is a high-performance RPC framework using HTTP/2 and Protocol Buffers (protobuf) for serialization.

---

## Comparison Table

| Dimension | REST (HTTP/1.1 + JSON) | gRPC (HTTP/2 + Protobuf) |
|-----------|------------------------|--------------------------|
| Protocol | HTTP/1.1 | HTTP/2 |
| Serialization | JSON (text) | Protobuf (binary) |
| Payload size | Large | 3–10× smaller |
| Speed | Slower | 5–10× faster |
| Streaming | No (SSE/WS needed) | Native (4 types) |
| Type safety | None (OpenAPI optional) | Strong (schema required) |
| Browser support | Full | Limited (gRPC-Web) |
| Human readable | Yes | No (binary) |
| Code generation | Optional | Required (from .proto) |
| Versioning | URL (v1, v2) | Field numbers in proto |

---

## gRPC Streaming Types

```protobuf
service ChatService {
  // Unary: 1 request → 1 response (like REST)
  rpc SendMessage (SendMessageRequest) returns (MessageResponse);
  
  // Server streaming: 1 request → stream of responses
  rpc GetMessageHistory (HistoryRequest) returns (stream Message);
  
  // Client streaming: stream of requests → 1 response
  rpc UploadMessages (stream Message) returns (UploadResponse);
  
  // Bidirectional streaming: both sides stream
  rpc Chat (stream ChatMessage) returns (stream ChatMessage);
}
```

---

## Protocol Buffer Definition

```protobuf
syntax = "proto3";

package com.example.payment;
option java_package = "com.example.payment.grpc";

message PaymentRequest {
  string order_id = 1;
  string user_id = 2;
  int64 amount_cents = 3;  // Using cents to avoid float precision
  string currency = 4;
  string payment_token = 5;
}

message PaymentResponse {
  string payment_id = 1;
  PaymentStatus status = 2;
  string error_message = 3;
  int64 processed_at_unix = 4;
}

enum PaymentStatus {
  UNKNOWN = 0;
  SUCCEEDED = 1;
  FAILED = 2;
  PENDING = 3;
}

service PaymentService {
  rpc ProcessPayment (PaymentRequest) returns (PaymentResponse);
  rpc GetPaymentStatus (GetStatusRequest) returns (stream PaymentStatusUpdate);
}
```

---

## Java: gRPC Server

```java
@GrpcService
public class PaymentGrpcService extends PaymentServiceGrpc.PaymentServiceImplBase {

    private final PaymentService paymentService;

    @Override
    public void processPayment(PaymentRequest request,
                               StreamObserver<PaymentResponse> responseObserver) {
        try {
            // Validate
            if (request.getAmountCents() <= 0) {
                responseObserver.onError(
                    Status.INVALID_ARGUMENT
                        .withDescription("Amount must be positive")
                        .asRuntimeException()
                );
                return;
            }
            
            // Process
            PaymentResult result = paymentService.process(
                request.getOrderId(),
                request.getUserId(),
                request.getAmountCents(),
                request.getCurrency()
            );
            
            // Response
            PaymentResponse response = PaymentResponse.newBuilder()
                .setPaymentId(result.getPaymentId())
                .setStatus(PaymentStatus.SUCCEEDED)
                .setProcessedAtUnix(result.getProcessedAt().getEpochSecond())
                .build();
            
            responseObserver.onNext(response);
            responseObserver.onCompleted();
            
        } catch (Exception e) {
            responseObserver.onError(
                Status.INTERNAL
                    .withDescription(e.getMessage())
                    .withCause(e)
                    .asRuntimeException()
            );
        }
    }

    // Server-streaming: real-time payment status updates
    @Override
    public void getPaymentStatus(GetStatusRequest request,
                                 StreamObserver<PaymentStatusUpdate> responseObserver) {
        String paymentId = request.getPaymentId();
        
        // Poll for status updates and stream to client
        ScheduledExecutorService executor = Executors.newSingleThreadScheduledExecutor();
        executor.scheduleAtFixedRate(() -> {
            PaymentStatus status = paymentService.getStatus(paymentId);
            responseObserver.onNext(PaymentStatusUpdate.newBuilder()
                .setPaymentId(paymentId)
                .setStatus(status)
                .build());
            
            if (status == PaymentStatus.SUCCEEDED || status == PaymentStatus.FAILED) {
                responseObserver.onCompleted();
                executor.shutdown();
            }
        }, 0, 2, TimeUnit.SECONDS);
    }
}
```

---

## Java: gRPC Client

```java
@Service
public class PaymentGrpcClient {

    private final PaymentServiceGrpc.PaymentServiceBlockingStub blockingStub;
    private final PaymentServiceGrpc.PaymentServiceStub asyncStub;

    public PaymentGrpcClient(@Value("${payment.service.host}") String host,
                              @Value("${payment.service.port}") int port) {
        ManagedChannel channel = ManagedChannelBuilder
            .forAddress(host, port)
            .useTransportSecurity() // TLS
            .build();
        
        this.blockingStub = PaymentServiceGrpc.newBlockingStub(channel)
            .withDeadlineAfter(3, TimeUnit.SECONDS); // Timeout
        this.asyncStub = PaymentServiceGrpc.newStub(channel);
    }

    public PaymentResult processPayment(Order order) {
        PaymentRequest request = PaymentRequest.newBuilder()
            .setOrderId(order.getOrderId())
            .setUserId(order.getUserId())
            .setAmountCents(order.getTotalCents())
            .setCurrency(order.getCurrency())
            .setPaymentToken(order.getPaymentToken())
            .build();
        
        try {
            PaymentResponse response = blockingStub.processPayment(request);
            return PaymentResult.from(response);
        } catch (StatusRuntimeException e) {
            if (e.getStatus().getCode() == Status.Code.DEADLINE_EXCEEDED) {
                throw new PaymentTimeoutException("Payment service timed out");
            }
            throw new PaymentServiceException(e.getStatus().getDescription());
        }
    }
}
```

---

## When to Use gRPC

| Use gRPC | Use REST |
|---------|---------|
| Microservice-to-microservice (internal) | Public-facing APIs |
| High-throughput, low-latency needs | Browser clients |
| Streaming data (real-time) | Simple CRUD |
| Polyglot environments (generate clients) | Human-readable debugging |
| Mobile clients (binary = smaller payloads) | Cache-friendly GET endpoints |

---

## AWS Mapping

| Component | AWS Support |
|-----------|------------|
| gRPC on ECS/EKS | Full support (ALB supports gRPC, HTTP/2) |
| gRPC on Lambda | Not native; use REST proxy |
| gRPC-Web (browser) | AWS ALB + envoy proxy |
| Service mesh | App Mesh (Envoy) supports gRPC |
| API Gateway | HTTP APIs support gRPC (limited) |

---

## Interview Q&A

**Q1: Why is gRPC faster than REST?**
> (1) HTTP/2: multiplexed streams on single TCP connection (no head-of-line blocking), binary framing, header compression (HPACK). (2) Protobuf: binary serialization is 3-10× smaller than JSON and faster to parse. (3) Streaming: data starts flowing before full message is received (reduces latency). Together: 5-10× faster than REST + JSON for same workload.

**Q2: What are the downsides of gRPC for public APIs?**
> (1) Browser support: gRPC requires HTTP/2 trailers which browsers don't expose — need gRPC-Web proxy. (2) Not human-readable: can't curl a gRPC endpoint and read the response. (3) Schema coupling: both client and server must share the .proto file. (4) Less cacheable: gRPC uses POST (like GraphQL). For public APIs targeting developers, REST is usually better.

**Q3: How do you version gRPC APIs?**
> Protobuf field numbers ensure backward compatibility: (1) Never remove or renumber existing fields. (2) Add new fields with new field numbers — old clients ignore them, new clients can use them. (3) For breaking changes: create a new service (v2) alongside the old one. (4) Mark old fields as deprecated. This allows rolling migration without coordinated deployments.
