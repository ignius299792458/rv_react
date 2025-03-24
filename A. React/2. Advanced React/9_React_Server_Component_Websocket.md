To make our **React Server Components (RSC) app even more powerful**, let's integrate **WebSockets** for real-time updates.  

Instead of **manual state refresh**, new users will appear in the **dashboard instantly** without reloading. 🚀  

---

# **🌟 Adding WebSockets for Real-Time Updates**
We'll use **Next.js API Routes** to create a WebSocket **server** and **client components** to listen for updates.

## **1️⃣ Setup WebSocket Server in Next.js**
🔹 Since **Next.js doesn’t provide WebSocket support out of the box**, we’ll use **`ws` (WebSocket library for Node.js)**.

### **Install WebSocket Library**
```bash
npm install ws
```

---

### **Create a WebSocket Server**
We’ll create a **server-side WebSocket instance** inside an API route.

```javascript
// 📌 app/api/socket/route.js
import { NextResponse } from "next/server";
import { WebSocketServer } from "ws";

let wss = null;

export function GET(req) {
  if (!wss) {
    wss = new WebSocketServer({ noServer: true });

    wss.on("connection", (ws) => {
      console.log("New WebSocket connection!");

      ws.on("message", (message) => {
        console.log("Received:", message);
      });
    });
  }

  return NextResponse.json({ message: "WebSocket server is running" });
}

export function broadcast(data) {
  if (wss) {
    wss.clients.forEach((client) => {
      if (client.readyState === 1) {
        client.send(JSON.stringify(data)); // Send message to all clients
      }
    });
  }
}
```

✅ **What This Does:**
- Initializes a WebSocket server **only once**.
- **Handles new connections** and listens for messages.
- **Broadcasts updates** to all connected clients.

---

## **2️⃣ Modify API to Broadcast New Users**
When a new user signs up, **broadcast the update** via WebSockets.

```javascript
// 📌 app/api/addUser/route.js
import { NextResponse } from "next/server";
import db from "../../lib/database"; // Simulated DB
import { broadcast } from "../socket/route"; // Import WebSocket broadcast function

export async function POST(req) {
  const { name, email } = await req.json();
  const newUser = await db.addUser({ name, email });

  // 🔥 Broadcast new user to all clients
  broadcast({ type: "NEW_USER", user: newUser });

  return NextResponse.json({ message: "User added", user: newUser });
}
```

✅ **Now, every time a user signs up**, the WebSocket **notifies all connected clients**.

---

## **3️⃣ Client Component to Listen for Updates**
🔹 We’ll create a **WebSocket client** that listens for real-time user updates.

```jsx
// 📌 app/dashboard/UserList.client.js
"use client";
import { useState, useEffect } from "react";

export default function UserList() {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    async function fetchUsers() {
      const res = await fetch("/api/users");
      const data = await res.json();
      setUsers(data);
    }

    fetchUsers();

    // 🌐 Connect to WebSocket server
    const ws = new WebSocket("ws://localhost:3000/api/socket");

    ws.onmessage = (event) => {
      const message = JSON.parse(event.data);

      if (message.type === "NEW_USER") {
        setUsers((prevUsers) => [...prevUsers, message.user]); // 🔥 Add new user dynamically
      }
    };

    return () => ws.close(); // Cleanup WebSocket connection
  }, []);

  return (
    <ul>
      {users.map((user) => (
        <li key={user.id}>{user.name} ({user.email})</li>
      ))}
    </ul>
  );
}
```

✅ **What This Does:**
- Fetches the initial user list from the API.
- **Connects to WebSocket server** and listens for updates.
- **Adds new users dynamically** when a message is received.

---

## **4️⃣ Updating the Dashboard to Use WebSockets**
Now, replace `UserData.server.js` with the **WebSocket-enabled `UserList.client.js`**.

```jsx
// 📌 app/dashboard/page.js
import AddUser from "./AddUser.client";
import UserList from "./UserList.client";

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      <AddUser />
      <UserList /> {/* 🔥 Now updates in real-time via WebSockets */}
    </div>
  );
}
```

---

## **🎯 Final Results**
✅ **No more manual refreshes** – Users appear **instantly**.  
✅ **WebSockets enable real-time updates**.  
✅ **Efficient mix of Server + Client Components**.  
✅ **Authentication remains secure via cookies**.  

🔥 Now we have a **full-stack React Server Components app** with **real-time updates via WebSockets!** 🚀  
