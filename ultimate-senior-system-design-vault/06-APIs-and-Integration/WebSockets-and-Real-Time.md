# WebSockets and Real-Time Communication

> **References:** [WebSocket RFC 6455](https://datatracker.ietf.org/doc/html/rfc6455) | [Socket.io Architecture](https://socket.io/docs/v4/how-it-works/) | [AWS API Gateway WebSocket](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-websocket-api.html)

---

## Real-Time Communication Options

| Technique | Direction | Protocol | Use Case |
|-----------|-----------|---------|---------|
| **Short Polling** | Client pulls | HTTP | Simple, works everywhere |
| **Long Polling** | Client holds conn | HTTP | Moderate real-time |
| **Server-Sent Events (SSE)** | Server → Client | HTTP/1.1 | Server push only |
| **WebSocket** | Bidirectional | TCP (WS/WSS) | Full duplex |
| **WebRTC** | Peer-to-peer | DTLS/SRTP | Video/audio, P2P data |

---

## Comparison Table

| Dimension | Polling | Long Polling | SSE | WebSocket |
|-----------|---------|-------------|-----|-----------|
| Direction | Client only | Client → Server only | Server only | Bidirectional |
| Overhead | High (repeated HTTP) | Medium | Low | Lowest |
| Latency | High (poll interval) | Low | Low | Lowest |
| Scalability | Easy | Medium | Easy | Hard (persistent conn) |
| Proxy support | Full | Full | Full | Sometimes blocked |
| Browser support | Full | Full | Full | Full (modern) |
| AWS | API Gateway REST | API Gateway REST | API Gateway HTTP | API Gateway WebSocket |
| Best for | Infrequent updates | Chat (fallback) | News feed, notifications | Chat, gaming, collaboration |

---

## WebSocket Deep Dive

### Connection Lifecycle
```
1. HTTP Upgrade handshake
   GET /chat HTTP/1.1
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
   
2. Server 101 Switching Protocols
   HTTP/1.1 101 Switching Protocols
   Upgrade: websocket
   Connection: Upgrade
   Sec-WebSocket-Accept: <hash>
   
3. Persistent bidirectional TCP tunnel
   Client → Server: text/binary frames
   Server → Client: text/binary frames
   Either side → ping/pong for keepalive
   Either side → close frame to terminate
```

---

## Java: WebSocket Server

```java
@Configuration
@EnableWebSocket
public class WebSocketConfig implements WebSocketConfigurer {

    private final ChatWebSocketHandler chatHandler;
    private final JwtHandshakeInterceptor jwtInterceptor;

    @Override
    public void registerWebSocketHandlers(WebSocketHandlerRegistry registry) {
        registry.addHandler(chatHandler, "/ws/chat")
            .addInterceptors(jwtInterceptor)
            .setAllowedOriginPatterns("https://*.example.com");
    }
}

@Component
public class ChatWebSocketHandler extends ConcurrentWebSocketSessionDecorator {

    // Thread-safe session registry
    private final ConcurrentHashMap<String, WebSocketSession> sessions = 
        new ConcurrentHashMap<>();
    private final ObjectMapper objectMapper = new ObjectMapper();

    @Override
    public void afterConnectionEstablished(WebSocketSession session) {
        String userId = getUserId(session);
        sessions.put(userId, session);
        
        log.info("WS connected: userId={}, sessionId={}", userId, session.getId());
        
        // Send pending messages if user was offline
        sendPendingMessages(userId, session);
    }

    @Override
    protected void handleTextMessage(WebSocketSession session, TextMessage message) throws Exception {
        WsEvent event = objectMapper.readValue(message.getPayload(), WsEvent.class);
        
        switch (event.getType()) {
            case "CHAT_MESSAGE" -> handleChatMessage(session, event);
            case "TYPING_START" -> broadcastTyping(session, event, true);
            case "TYPING_STOP" -> broadcastTyping(session, event, false);
            case "PING" -> session.sendMessage(new TextMessage("{\"type\":\"PONG\"}"));
        }
    }

    private void handleChatMessage(WebSocketSession senderSession, WsEvent event) throws IOException {
        String senderId = getUserId(senderSession);
        String recipientId = event.getData().get("recipientId").asText();
        String content = event.getData().get("content").asText();
        
        // Deliver to recipient if online
        WebSocketSession recipientSession = sessions.get(recipientId);
        if (recipientSession != null && recipientSession.isOpen()) {
            WsEvent delivery = WsEvent.chatMessage(senderId, content);
            recipientSession.sendMessage(new TextMessage(
                objectMapper.writeValueAsString(delivery)));
        } else {
            // Offline — queue for push notification
            offlineDeliveryService.queue(recipientId, content);
        }
    }

    @Override
    public void afterConnectionClosed(WebSocketSession session, CloseStatus status) {
        String userId = getUserId(session);
        sessions.remove(userId);
        presenceService.setOffline(userId);
        log.info("WS disconnected: userId={}, status={}", userId, status);
    }

    public void sendToUser(String userId, Object message) {
        WebSocketSession session = sessions.get(userId);
        if (session != null && session.isOpen()) {
            try {
                session.sendMessage(new TextMessage(objectMapper.writeValueAsString(message)));
            } catch (IOException e) {
                sessions.remove(userId);
            }
        }
    }
    
    private String getUserId(WebSocketSession session) {
        return (String) session.getAttributes().get("userId");
    }
}
```

---

## React Frontend: WebSocket Client

```jsx
import { useEffect, useRef, useCallback, useState } from 'react';

function useChatWebSocket(roomId, token) {
  const wsRef = useRef(null);
  const [messages, setMessages] = useState([]);
  const [connected, setConnected] = useState(false);
  const reconnectTimeoutRef = useRef(null);

  const connect = useCallback(() => {
    const ws = new WebSocket(`wss://api.example.com/ws/chat?token=${token}&room=${roomId}`);
    wsRef.current = ws;

    ws.onopen = () => {
      setConnected(true);
      console.log('WebSocket connected');
    };

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);
      if (message.type === 'CHAT_MESSAGE') {
        setMessages(prev => [...prev, message]);
      }
    };

    ws.onclose = (event) => {
      setConnected(false);
      if (!event.wasClean) {
        // Auto-reconnect with exponential backoff
        const delay = Math.min(1000 * Math.pow(2, reconnectAttempts), 30000);
        reconnectTimeoutRef.current = setTimeout(connect, delay);
      }
    };

    ws.onerror = (error) => {
      console.error('WebSocket error:', error);
    };
  }, [roomId, token]);

  const sendMessage = useCallback((content) => {
    if (wsRef.current?.readyState === WebSocket.OPEN) {
      wsRef.current.send(JSON.stringify({
        type: 'CHAT_MESSAGE',
        data: { content }
      }));
    }
  }, []);

  useEffect(() => {
    connect();
    return () => {
      wsRef.current?.close(1000, 'Component unmounted');
      clearTimeout(reconnectTimeoutRef.current);
    };
  }, [connect]);

  return { messages, connected, sendMessage };
}
```

---

## Server-Sent Events (SSE) — Server Push Only

```java
@RestController
public class NotificationController {

    private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

    // Client subscribes to server push stream
    @GetMapping(value = "/notifications/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public SseEmitter subscribe(@RequestHeader("X-User-Id") String userId) {
        SseEmitter emitter = new SseEmitter(0L); // No timeout
        emitters.put(userId, emitter);
        
        emitter.onCompletion(() -> emitters.remove(userId));
        emitter.onTimeout(() -> emitters.remove(userId));
        
        return emitter;
    }

    // Push event to user
    public void pushNotification(String userId, Notification notification) {
        SseEmitter emitter = emitters.get(userId);
        if (emitter != null) {
            try {
                emitter.send(SseEmitter.event()
                    .id(notification.getId())
                    .name("NOTIFICATION")
                    .data(objectMapper.writeValueAsString(notification)));
            } catch (IOException e) {
                emitters.remove(userId);
            }
        }
    }
}
```

---

## AWS Deployment

| Option | AWS Service | Scale |
|--------|------------|-------|
| Managed WebSocket | API Gateway WebSocket | 500K connections |
| Custom WebSocket | ECS/EKS + ALB (WebSocket supported) | Unlimited |
| SSE | ALB + ECS | High |
| Real-time notifications | AppSync subscriptions | High |
| Push (offline) | SNS Mobile Push | Unlimited |

---

## Interview Q&A

**Q1: How do WebSockets differ from HTTP long-polling?**
> Long-polling: client sends HTTP request, server holds it open (up to timeout), sends response when data is available, client immediately reconnects. Overhead: HTTP headers on every round trip, connection setup/teardown. WebSocket: one HTTP upgrade handshake, then persistent TCP tunnel. No HTTP overhead per message. WebSocket is 10× more efficient for high-frequency bidirectional communication.

**Q2: How do you scale WebSocket servers?**
> WebSocket servers are stateful (connections are sticky). Scale with: (1) Sticky sessions at ALB (route same user to same server). (2) Redis pub/sub for cross-server message delivery (when A and B are on different servers). (3) Horizontal scaling by adding more WS server instances. (4) AWS API Gateway WebSocket: managed, auto-scales, but has connection limits.

**Q3: When would you choose SSE over WebSocket?**
> SSE is better when: (1) Communication is server-to-client only (news feeds, stock prices, notifications). (2) You need auto-reconnect for free (SSE spec includes it). (3) Proxies/firewalls block WebSocket upgrades. (4) You want to use HTTP/2 multiplexing. WebSocket for: bidirectional communication (chat, gaming), low-latency (<10ms) requirements, binary data streaming.
