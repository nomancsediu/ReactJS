# Real-Time Data

Some apps need live updates. Chat messages appear instantly. Stock prices change every second. Notifications pop up the moment something happens. I used to reload the page to see new data. There is a better way.

## Polling: The Simple Approach

Polling means asking the server for updates at regular intervals. It is easy to implement:

```jsx
function LiveScore({ gameId }) {
  const [score, setScore] = useState(null);

  useEffect(() => {
    // Fetch immediately
    function fetchScore() {
      fetch(`/api/games/${gameId}/score`)
        .then((res) => res.json())
        .then(setScore);
    }

    fetchScore();
    // Then poll every 5 seconds
    const interval = setInterval(fetchScore, 5000);
    return () => clearInterval(interval);
  }, [gameId]);

  return <div className="text-4xl font-bold">{score?.home} - {score?.away}</div>;
}
```

Polling works but it wastes resources. Most requests return the same data. Use it for simple cases where real-time precision is not critical.

## WebSocket: True Real-Time

WebSockets keep a persistent connection open. The server pushes updates instantly:

```jsx
function useWebSocket(url) {
  const [data, setData] = useState(null);
  const [status, setStatus] = useState("connecting");

  useEffect(() => {
    const ws = new WebSocket(url);

    ws.onopen = () => setStatus("connected");
    ws.onmessage = (event) => setData(JSON.parse(event.data));
    ws.onerror = () => setStatus("error");
    ws.onclose = () => setStatus("disconnected");

    return () => ws.close();
  }, [url]);

  return { data, status };
}

// Usage
function ChatRoom({ roomId }) {
  const { data: message, status } = useWebSocket(
    `wss://api.example.com/chat/${roomId}`
  );
  const [messages, setMessages] = useState([]);

  useEffect(() => {
    if (message) setMessages((prev) => [...prev, message]);
  }, [message]);

  return (
    <div>
      <span className="text-sm text-gray-500">{status}</span>
      {messages.map((msg, i) => <ChatBubble key={i} message={msg} />)}
    </div>
  );
}
```

## Socket.io Client

Socket.io adds reconnection, rooms, and events on top of WebSocket:

```jsx
import { io } from "socket.io-client";

function useSocket(url) {
  const [socket, setSocket] = useState(null);

  useEffect(() => {
    const s = io(url, { transports: ["websocket"] });

    s.on("connect", () => console.log("Connected"));
    s.on("disconnect", () => console.log("Disconnected"));

    setSocket(s);
    return () => s.disconnect();
  }, [url]);

  return socket;
}

function LiveNotifications() {
  const socket = useSocket("https://api.example.com");

  useEffect(() => {
    if (!socket) return;
    socket.on("notification", (data) => {
      showToast(data.message);
    });
    return () => socket.off("notification");
  }, [socket]);

  return <NotificationList />;
}
```

## Choosing the Right Approach

| Approach | Best For | Latency |
|----------|----------|---------|
| Polling | Simple, infrequent updates | Seconds |
| WebSocket | Low-latency, two-way data | Milliseconds |
| Socket.io | When you need reliability features | Milliseconds |

I start with polling for prototypes. When the app needs instant updates, I switch to WebSocket or Socket.io. Do not over-engineer on day one.
