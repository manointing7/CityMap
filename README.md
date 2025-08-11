
---

```markdown
CityMap

*CityMap* is an AI-powered super app combining ride-hailing, an AI co-pilot with live maps and traffic, food delivery (*CityEats*), and courier services (*CityDispatch*).

---

🚀 Features

- Real-time ride booking and tracking
- AI assistant for navigation and traffic updates
- Google Maps integration
- Modular expansion: CityEats, CityDispatch

---

🧱 Tech Stack

- *Frontend*: React (Vite), Google Maps API, OpenAI API
- *Backend*: Node.js + Express
- *Database*: MongoDB Atlas (future use)
- *AI*: OpenAI GPT-3.5 API

---

📁 Folder Structure

```
CityMap/
├── client/          → React frontend
│   ├── index.html
│   └── src/
│       ├── App.jsx
│       └── main.jsx
├── server/          → Express backend
│   ├── index.js
│   └── .env
└── README.md
```

---

⚙️ Setup Instructions

1. Clone the repo

```bash
git clone https://github.com/yourusername/CityMap.git
cd CityMap
```

---

2. Setup Frontend

```bash
cd client
npm create vite@latest . -- --template react
npm install
npm install @react-google-maps/api
```

Replace `/client/src/App.jsx` with:

```jsx

import { useLoadScript, GoogleMap, Marker } from "@react-google-maps/api";
import { useState } from "react";

const center = { lat: 6.5244, lng: 3.3792 }; // Lagos example

export default function App() {
  const [message, setMessage] = useState("");
  const [reply, setReply] = useState("");

  const { isLoaded } = useLoadScript({
    googleMapsApiKey: "YOUR_GOOGLE_MAPS_API_KEY",
  });

  async function askAI() {
    const res = await fetch("http://localhost:5000/ai", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ prompt: message }),
    });
    const data = await res.json();
    setReply(data.reply);
  }

  if (!isLoaded) return <div>Loading Maps...</div>;

  return (
    <div style={{ padding: 20 }}>
      <h1>CityMap AI Co-Pilot</h1>
      <GoogleMap
        center={center}
        zoom={12}
        mapContainerStyle={{ width: "100%", height: "300px", marginBottom: "1rem" }}
      >
        <Marker position={center} />
      </GoogleMap>

      <textarea
        rows={4}
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Ask AI for traffic updates"
        style={{ width: "100%" }}
      />
      <button onClick={askAI} style={{ marginTop: 10 }}>
        Ask AI

</button>
      {reply && (
        <div style={{ marginTop: 20 }}>
          <strong>AI Reply:</strong>
          <p>{reply}</p>
        </div>
      )}
    </div>
  );
}
```

Replace `/client/src/main.jsx` with:

```jsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

ReactDOM.createRoot(document.getElementById("root")).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

Run the frontend:

```bash
npm run dev
```

---

3. Setup Backend

```bash
cd ../server
npm init -y
npm install express cors dotenv openai
```

Create `index.js` inside `/server`:

```js
import express from 'express';
import cors from 'cors';
import dotenv from 'dotenv';
import { Configuration, OpenAIApi } from 'openai';

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const config = new Configuration({
  apiKey: process.env.OPENAI_API_KEY,
});
const openai = new OpenAIApi(config);

app.post('/ai', async (req, res) => {
  const { prompt } = req.body;
  try {
    const response = await openai.createChatCompletion({
      model: 'gpt-3.5-turbo',
      messages: [{ role: "user", content: prompt }],
    });
    res.json({ reply: response.data.choices[0].message.content });
  } catch (err) {

res.status(500).json({ error: "AI error", details: err.message });
  }
});

app.get('/', (req, res) => res.send("CityMap backend is live"));

const PORT = process.env.PORT || 5000;
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

Create `.env` file:

```
OPENAI_API_KEY=your_openai_key
PORT=5000
```

Run the backend:

```bash
node index.js
```

---

✅ Final Step

- Open frontend at http://localhost:5173
- Backend runs at http://localhost:5000
- Ask the AI anything traffic-related + view live Google Map

---

🔐 Notes

- Replace API keys with your own (never push them to GitHub).
- You can expand this into CityEats and CityDispatch later.

---

👏 Credits

Built by [Glowstar Innovative Tech Solutions] • Inspired by Uber + AI + Map integration
```

---
- *Backend*: Node.js + Express + MongoDB
- *Auth*: JWT tokens
- *Password security*: bcrypt

---

✅ Backend Setup for Auth

*1. Install extra packages*
In `/server`:

```bash
npm install bcryptjs jsonwebtoken mongoose
```

---

*2. Create `/server/models/User.js`*

```js
import mongoose from 'mongoose';

const UserSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
});

export default mongoose.model('User', UserSchema);
```

---

*3. Add auth routes to `index.js`*

Update your `/server/index.js`:

```js
import mongoose from 'mongoose';
import bcrypt from 'bcryptjs';
import jwt from 'jsonwebtoken';
import User from './models/User.js'; // <-- add this line

mongoose.connect(process.env.MONGO_URI);

// Register
app.post('/register', async (req, res) => {
  const { name, email, password } = req.body;
  const hashed = await bcrypt.hash(password, 10);
  try {
    const user = await User.create({ name, email, password: hashed });
    res.json({ message: "User registered", user });
  } catch (err) {
    res.status(400).json({ error: "Email already exists" });
  }
});

// Login
app.post('/login', async (req, res) => {

const { email, password } = req.body;
  const user = await User.findOne({ email });
  if (!user) return res.status(400).json({ error: "User not found" });

  const isMatch = await bcrypt.compare(password, user.password);
  if (!isMatch) return res.status(401).json({ error: "Invalid password" });

  const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET);
  res.json({ message: "Login successful", token });
});
```

---

*4. Add to `.env`*
```
MONGO_URI=your_mongo_uri
JWT_SECRET=your_random_secret
```

---

✅ Now your backend supports:
- `POST /register`
- `POST /login`

---

✅ Step 1: Create `Auth.jsx` in `/client/src`

```jsx
import { useState } from "react";

export default function Auth() {
  const [isLogin, setIsLogin] = useState(true);
  const [form, setForm] = useState({ name: "", email: "", password: "" });
  const [message, setMessage] = useState("");

  const toggle = () => {
    setIsLogin(!isLogin);
    setMessage("");
    setForm({ name: "", email: "", password: "" });
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    const url = isLogin ? "login" : "register";

    const res = await fetch(`http://localhost:5000/${url}`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(form),
    });

    const data = await res.json();
    if (res.ok) {
      setMessage(data.message);
      if (isLogin) localStorage.setItem("token", data.token);
    } else {
      setMessage(data.error);
    }
  };

  return (
    <div style={{ padding: 20 }}>
      <h2>{isLogin ? "Login" : "Register"} to CityMap</h2>
      <form onSubmit={handleSubmit}>
        {!isLogin && (
          <input
            type="text"
            placeholder="Name"
            value={form.name}

onChange={(e) => setForm({ ...form, name: e.target.value })}
            required
          />
        )}
        <br />
        <input
          type="email"
          placeholder="Email"
          value={form.email}
          onChange={(e) => setForm({ ...form, email: e.target.value })}
          required
        />
        <br />
        <input
          type="password"
          placeholder="Password"
          value={form.password}
          onChange={(e) => setForm({ ...form, password: e.target.value })}
          required
        />
        <br />
        <button type="submit">{isLogin ? "Login" : "Register"}</button>
      </form>
      <p>{message}</p>
      <button onClick={toggle}>
        {isLogin ? "Need an account?" : "Already have an account?"}
      </button>
    </div>
  );
}
```

---

✅ Step 2: Use it in `App.jsx`

Replace your current `App.jsx` content with this to include both Auth + Map:

```jsx
import Auth from "./Auth";
import { useState } from "react";
import { GoogleMap, useLoadScript, Marker } from "@react-google-maps/api";

const center = { lat: 6.5244, lng: 3.3792 }; // Lagos

export default function App() {
  const [message, setMessage] = useState("");
  const [reply, setReply] = useState("");

const [isAuth, setIsAuth] = useState(() => !!localStorage.getItem("token"));

  const { isLoaded } = useLoadScript({
    googleMapsApiKey: "YOUR_GOOGLE_MAPS_API_KEY",
  });

  async function askAI() {
    const res = await fetch("http://localhost:5000/ai", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ prompt: message }),
    });
    const data = await res.json();
    setReply(data.reply);
  }

  if (!isAuth) return <Auth />;

  if (!isLoaded) return <div>Loading Map...</div>;

  return (
    <div style={{ padding: 20 }}>
      <h1>Welcome to CityMap</h1>
      <GoogleMap
        center={center}
        zoom={12}
        mapContainerStyle={{ width: "100%", height: "300px" }}
      >
        <Marker position={center} />
      </GoogleMap>
      <textarea
        rows={3}
        value={message}
        onChange={(e) => setMessage(e.target.value)}
        placeholder="Ask AI about traffic..."
      />
      <button onClick={askAI}>Ask AI</button>
      <p>{reply}</p>
    </div>
  );
}
```

---

---

✅ 1. Update Backend User Model

In `/server/models/User.js`, add `role`:

```js
const UserSchema = new mongoose.Schema({
  name: String,
  email: { type: String, unique: true },
  password: String,
  role: { type: String, enum: ['user', 'driver'], default: 'user' },
});
```

---

✅ 2. Update `/register` route to accept role

In `index.js`:

```js
const { name, email, password, role } = req.body;
const user = await User.create({ name, email, password: hashed, role });
```

---

✅ 3. Add auth middleware

Create `/server/middleware/auth.js`:

```js
import jwt from 'jsonwebtoken';

export function verifyToken(req, res, next) {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(401).json({ error: "Missing token" });

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ error: "Invalid token" });
  }
}
```

---

✅ 4. Create a protected route

In `index.js`:

```js
import { verifyToken } from './middleware/auth.js';

app.get('/profile', verifyToken, async (req, res) => {
  const user = await User.findById(req.user.id).select("-password");
  res.json(user);

});
```

---

✅ 5. Update Frontend Auth to include role & logout

In `Auth.jsx`, update the form:

```jsx
{!isLogin && (
  <>
    <input
      type="text"
      placeholder="Name"
      value={form.name}
      onChange={(e) => setForm({ ...form, name: e.target.value })}
    /><br/>
    <select
      value={form.role}
      onChange={(e) => setForm({ ...form, role: e.target.value })}
    >
      <option value="user">User</option>
      <option value="driver">Driver</option>
    </select><br/>
  </>
)}
```

In your `App.jsx`, add a logout button:

```jsx
<button onClick={() => {
  localStorage.removeItem("token");
  window.location.reload();
}}>Logout</button>
```

---

Now:
- Users can register as "user" or "driver"
- JWT tokens protect routes
- You can customize pages based on roles

Perfect — here’s how to build a simple *Driver Dashboard* to show:

- Driver’s info (name, email, role)
- Future: Active rides, earnings, availability toggle

---

✅ 1. Create `DriverDashboard.jsx` in `/client/src`

```jsx
import { useEffect, useState } from "react";

export default function DriverDashboard() {
  const [driver, setDriver] = useState(null);

  useEffect(() => {
    const fetchDriver = async () => {
      const token = localStorage.getItem("token");
      const res = await fetch("http://localhost:5000/profile", {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      });
      const data = await res.json();
      if (data.role === "driver") {
        setDriver(data);
      } else {
        alert("Access denied: Not a driver");
        window.location.reload();
      }
    };
    fetchDriver();
  }, []);

  if (!driver) return <p>Loading driver dashboard...</p>;

  return (
    <div style={{ padding: 20 }}>
      <h2>Driver Dashboard</h2>
      <p><strong>Name:</strong> {driver.name}</p>
      <p><strong>Email:</strong> {driver.email}</p>
      <p><strong>Role:</strong> {driver.role}</p>

      {/* Future: add ride requests, earnings, toggle status */}
    </div>
  );
}
```

---

✅ 2. Show it in `App.jsx` (optional basic logic)

“`jsx
import Auth from "./Auth";
import DriverDashboard from "./DriverDashboard";
import  useEffect, useState  from "react";

export default function App() 
  const [user, setUser] = useState(null);

  useEffect(() => 
    const token = localStorage.getItem("token");
    if (!token) return;

    fetch("http://localhost:5000/profile", 
      headers:  Authorization: `Bearer{token}` },
    })
      .then(res => res.json())
      .then(data => setUser(data))
      .catch(() => localStorage.removeItem("token"));
  }, []);

  if (!user) return <Auth />;

  if (user.role === "driver") return <DriverDashboard />;

  return (
    <div style={{ padding: 20 }}>
      <h1>Welcome {user.name} to CityMap</h1>
      <p>Role: {user.role}</p>
      {/* Show Google Map and AI here for regular users */}
      <button onClick={() => {
        localStorage.removeItem("token");
        window.location.reload();
      }}>Logout</button>
    </div>
  );
}
```

---

This gives:
- Driver-only dashboard
- Protected access
- Simple role-based routing

---

✅ 1. Create `CityEats.jsx` in `/client/src`

```jsx
import { useEffect, useState } from "react";

const dummyRestaurants = [
  { name: "Suya Spot", food: "Grilled beef", location: "Yaba" },
  { name: "ChopLife Kitchen", food: "Jollof Rice", location: "Lekki" },
  { name: "Mama Put", food: "Amala & Ewedu", location: "Ikeja" },
];

export default function CityEats() {
  const [restaurants, setRestaurants] = useState([]);

  useEffect(() => {
    // In real app, fetch from backend
    setRestaurants(dummyRestaurants);
  }, []);

  return (
    <div style={{ padding: 20 }}>
      <h2>CityEats — Discover Local Food</h2>
      {restaurants.map((r, i) => (
        <div key={i} style={{ border: "1px solid #ccc", margin: 10, padding: 10 }}>
          <h4>{r.name}</h4>
          <p>{r.food}</p>
          <p><strong>Location:</strong> {r.location}</p>
          <button>Order</button>
        </div>
      ))}
    </div>
  );
}
```

---

✅ 2. Show it in `App.jsx` for users

In `App.jsx`, import it:

```js
import CityEats from "./CityEats";
```

Update inside the user section:

```jsx
return (
  <div style={{ padding: 20 }}>
    <h1>Welcome {user.name} to CityMap</h1>
    <p>Role: {user.role}</p>

<CityEats />

    <button onClick={() => {
      localStorage.removeItem("token");
      window.location.reload();
    }}>Logout</button>
  </div>
);
```

---

This gives:
- A mock food discovery UI
- Easy to extend to real APIs later
- Seamlessly integrates with your auth system

---

✅ 1. Create `CityDispatch.jsx` in `/client/src`

```jsx
import { useState } from "react";

export default function CityDispatch() {
  const [form, setForm] = useState({
    pickup: "",
    dropoff: "",
    parcelDetails: "",
  });

  const [message, setMessage] = useState("");

  const handleSubmit = (e) => {
    e.preventDefault();
    // For now, just simulate a submission
    setMessage(
      `Parcel scheduled from form.pickup to{form.dropoff}. Details: ${form.parcelDetails}`
    );
    setForm({ pickup: "", dropoff: "", parcelDetails: "" });
  };

  return (
    <div style={{ padding: 20 }}>
      <h2>CityDispatch — Parcel Pickup & Drop-off</h2>
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          placeholder="Pickup Location"
          value={form.pickup}
          onChange={(e) => setForm({ ...form, pickup: e.target.value })}
          required
        />
        <br />
        <input
          type="text"
          placeholder="Drop-off Location"
          value={form.dropoff}
          onChange={(e) => setForm({ ...form, dropoff: e.target.value })}
          required
        />
        <br />
        <textarea
          placeholder="Parcel Details"

value={form.parcelDetails}
          onChange={(e) => setForm({ ...form, parcelDetails: e.target.value })}
          required
          rows={3}
        />
        <br />
        <button type="submit">Schedule Pickup</button>
      </form>
      <p>{message}</p>
    </div>
  );
}
```

---

✅ 2. Import & show it in `App.jsx` for users

At the top of `App.jsx`:

```js
import CityDispatch from "./CityDispatch";
```

In the user section’s return, add below CityEats:

```jsx
<CityEats />
<CityDispatch />
```

---

This creates:

- Simple form for parcel pickup/dropoff
- Immediate confirmation message
- Foundation to connect with backend dispatch/order logic later

---

---

✅ 1. Create `models/DispatchOrder.js` in `/server/models`

```js
import mongoose from 'mongoose';

const DispatchOrderSchema = new mongoose.Schema({
  userId: { type: mongoose.Schema.Types.ObjectId, ref: 'User' },
  pickup: String,
  dropoff: String,
  parcelDetails: String,
  createdAt: { type: Date, default: Date.now },
});

export default mongoose.model('DispatchOrder', DispatchOrderSchema);
```

---

✅ 2. Add routes in `/server/index.js`

```js
import DispatchOrder from './models/DispatchOrder.js';
import { verifyToken } from './middleware/auth.js';

// Create dispatch order
app.post('/dispatch', verifyToken, async (req, res) => {
  const { pickup, dropoff, parcelDetails } = req.body;
  try {
    const order = await DispatchOrder.create({
      userId: req.user.id,
      pickup,
      dropoff,
      parcelDetails,
    });
    res.json({ message: 'Order created', order });
  } catch (err) {
    res.status(500).json({ error: 'Failed to create order' });
  }
});

// Get user's dispatch orders
app.get('/dispatch', verifyToken, async (req, res) => {
  try {
    const orders = await DispatchOrder.find({ userId: req.user.id });
    res.json(orders);
  } catch {

res.status(500).json({ error: 'Failed to get orders' });
  }
});
```

---

✅ 3. Update `CityDispatch.jsx` frontend to call backend

Replace `handleSubmit` in `CityDispatch.jsx` with:

```jsx
const handleSubmit = async (e) => {
  e.preventDefault();
  const token = localStorage.getItem("token");

  const res = await fetch("http://localhost:5000/dispatch", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      Authorization: `Bearer token`,
    ,
    body: JSON.stringify(form),
  );
  const data = await res.json();

  if (res.ok) 
    setMessage("Order scheduled successfully!");
    setForm( pickup: "", dropoff: "", parcelDetails: "" );
   else 
    setMessage(data.error || "Failed to schedule order");
  ;
“`

—

✅ 4. (Optional) Show user's dispatch orders

Add this in `CityDispatch.jsx`:

“`jsx
import  useEffect  from "react";

const [orders, setOrders] = useState([]);

useEffect(() => 
  const fetchOrders = async () => 
    const token = localStorage.getItem("token");
    const res = await fetch("http://localhost:5000/dispatch", 
      headers:  Authorization: `Bearer{token}` },
    });
    const data = await res.json();
    if (res.ok) setOrders(data);
  };
  fetchOrders();
}, []);
```

And below the form, show:

```jsx

<h3>Your Dispatch Orders</h3>
<ul>
  {orders.map((o) => (
    <li key={o._id}>
      From {o.pickup} to {o.dropoff} — {o.parcelDetails}
    </li>
  ))}
</ul>
```

---

- Google Maps on frontend  
- Backend socket connection for real-time updates (we’ll use Socket.IO)  
- Driver sends location updates; user sees live ride position  

---

Step 1: Add Socket.IO to backend

Install:

```bash
npm install socket.io
```

Update `/server/index.js`:

```js
import http from "http";
import { Server } from "socket.io";

const server = http.createServer(app);
const io = new Server(server, {
  cors: { origin: "*" },
});

io.on("connection", (socket) => {
  console.log("User connected", socket.id);

  socket.on("joinRide", (rideId) => {
    socket.join(rideId);
    console.log(`Joined ride room rideId`);
  );

  socket.on("locationUpdate", ( rideId, lat, lng ) => 
    // Broadcast to everyone in the ride room except sender
    socket.to(rideId).emit("driverLocation",  lat, lng );
  );

  socket.on("disconnect", () => 
    console.log("User disconnected", socket.id);
  );
);

const PORT = process.env.PORT || 5000;
server.listen(PORT, () => console.log(`Server running on port{PORT}`));
```

---

Step 2: Frontend setup — install socket.io-client

```bash
npm install socket.io-client
```

---

Step 3: Create `LiveRide.jsx`

```jsx
import { useEffect, useState } from "react";

import { io } from "socket.io-client";
import { GoogleMap, Marker, useLoadScript } from "@react-google-maps/api";

const socket = io("http://localhost:5000");
const center = { lat: 6.5244, lng: 3.3792 }; // Default center

export default function LiveRide({ rideId }) {
  const [driverPos, setDriverPos] = useState(center);
  const { isLoaded } = useLoadScript({
    googleMapsApiKey: "YOUR_GOOGLE_MAPS_API_KEY",
  });

  useEffect(() => {
    socket.emit("joinRide", rideId);

    socket.on("driverLocation", (pos) => {
      setDriverPos(pos);
    });

    return () => {
      socket.off("driverLocation");
    };
  }, [rideId]);

  if (!isLoaded) return <div>Loading Map...</div>;

  return (
    <div>
      <h3>Live Ride Tracking</h3>
      <GoogleMap
        zoom={14}
        center={driverPos}
        mapContainerStyle={{ width: "100%", height: "300px" }}
      >
        <Marker position={driverPos} />
      </GoogleMap>
    </div>
  );
}
```

---

Step 4: Driver sends location updates

In driver’s app (example in React):

```jsx
import { useEffect } from "react";

function DriverLive({ rideId }) {
  useEffect(() => {
    const interval = setInterval(() => {
      navigator.geolocation.getCurrentPosition(({ coords }) => {
        socket.emit("locationUpdate", {
          rideId,

lat: coords.latitude,
          lng: coords.longitude,
        });
      });
    }, 3000); // every 3 seconds

    return () => clearInterval(interval);
  }, [rideId]);

  return <div>Sending live location updates...</div>;
}
```

---

Summary

- Backend socket.io server to handle connections & relay location  
- User’s frontend listens for location updates and shows on Google Map  
- Driver’s frontend sends live GPS positions regularly  

---

---

1. Install needed libs

Run in your `/client` folder:

```
npm install socket.io-client @react-google-maps/api
```

---

2. Create a new file `LiveRide.jsx` in `/client/src`

```jsx
import { useEffect, useState } from "react";
import { io } from "socket.io-client";
import { GoogleMap, Marker, useLoadScript } from "@react-google-maps/api";

const socket = io("http://localhost:5000");
const defaultCenter = { lat: 6.5244, lng: 3.3792 }; // Lagos center example

export default function LiveRide({ rideId }) {
  const [driverPos, setDriverPos] = useState(defaultCenter);
  const { isLoaded } = useLoadScript({
    googleMapsApiKey: "YOUR_GOOGLE_MAPS_API_KEY",
  });

  useEffect(() => {
    socket.emit("joinRide", rideId);

    socket.on("driverLocation", (pos) => {
      setDriverPos(pos);
    });

    return () => {
      socket.off("driverLocation");
    };
  }, [rideId]);

  if (!isLoaded) return <div>Loading Map...</div>;

  return (
    <div>
      <h3>Live Ride Tracking</h3>
      <GoogleMap
        zoom={14}
        center={driverPos}
        mapContainerStyle={{ width: "100%", height: "300px" }}
      >
        <Marker position={driverPos} />
      </GoogleMap>

</div>
  );
}
```

---

3. Create `DriverLive.jsx` to send driver location updates

```jsx
import { useEffect } from "react";
import { io } from "socket.io-client";

const socket = io("http://localhost:5000");

export default function DriverLive({ rideId }) {
  useEffect(() => {
    const sendLocation = () => {
      navigator.geolocation.getCurrentPosition(({ coords }) => {
        socket.emit("locationUpdate", {
          rideId,
          lat: coords.latitude,
          lng: coords.longitude,
        });
      });
    };

    sendLocation();
    const interval = setInterval(sendLocation, 3000); // every 3 sec

    return () => clearInterval(interval);
  }, [rideId]);

  return <div>Sending live location updates for ride {rideId}...</div>;
}
```

---

4. Use in your `App.jsx` or relevant components

Example usage for driver and user:

```jsx
import LiveRide from "./LiveRide";
import DriverLive from "./DriverLive";

export default function App() {
  // For demo, hardcoding rideId and user role
  const user = { role: "user" }; // or "driver"
  const rideId = "ride123";

  if (user.role === "driver") {
    return <DriverLive rideId={rideId} />;
  }

  if (user.role === "user") {
    return <LiveRide rideId={rideId} />;
  }

  return <div>Please log in</div>;
}
```

---

5. Make sure your backend server is running with the Socket.IO setup

Your backend server should run `server.listen(...)` on the `http` server wrapping your Express app, as shown previously.

---

This setup will:

- Driver sends GPS location every 3 seconds  
- User sees the driver's location live on the map  

---

- Receiving WhatsApp messages  
- Sending replies  
- Integrating AI for understanding user intent  

---

1. Setup: Install needed packages

```bash
npm install express twilio dotenv openai body-parser
```

---

2. Create `.env` file with:

```
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_WHATSAPP_NUMBER=whatsapp:+your_twilio_whatsapp_number
OPENAI_API_KEY=your_openai_api_key
PORT=4000
```

---

3. Create `whatsappBot.js`

```js
import express from "express";
import bodyParser from "body-parser";
import { config } from "dotenv";
import { Twilio } from "twilio";
import OpenAI from "openai";

config();

const app = express();
app.use(bodyParser.urlencoded({ extended: false }));

const client = new Twilio(process.env.TWILIO_ACCOUNT_SID, process.env.TWILIO_AUTH_TOKEN);
const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });

app.post("/whatsapp", async (req, res) => {
  const from = req.body.From; // WhatsApp sender number
  const msg = req.body.Body; // Incoming message text

  console.log(`Message from from:{msg}`);
// Call OpenAI to generate response (simple prompt)
  const completion = await openai.chat.completions.create({
    model: "gpt-4o-mini",
    messages: [
      { role: "system", content: "You are CityMap AI assistant. Help user book rides, order food, and get info." },
      { role: "user", content: msg },
    ],
  });

  const reply = completion.choices[0].message.content;

  // Send WhatsApp reply via Twilio
  await client.messages.create({
    from: process.env.TWILIO_WHATSAPP_NUMBER,
    to: from,
    body: reply,
  });

  res.sendStatus(200);
});

const PORT = process.env.PORT || 4000;
app.listen(PORT, () => console.log(`WhatsApp bot running on port ${PORT}`));
```

---

4. How to use

- Run this backend service: `node whatsappBot.js`  
- Set your Twilio WhatsApp webhook to point to `https://yourdomain.com/whatsapp` (or `http://localhost:4000/whatsapp` if testing locally with tools like ngrok)  
- When users send WhatsApp messages to your Twilio WhatsApp number, this bot replies using AI  

---

---

1. Install mongoose

```bash
npm install mongoose
```

---

2. Setup MongoDB connection in `whatsappBot.js` (add at top):

```js
import mongoose from "mongoose";

mongoose.connect("mongodb://localhost:27017/citymap", {
  useNewUrlParser: true,
  useUnifiedTopology: true,
}).then(() => console.log("MongoDB connected"))
  .catch(err => console.error(err));

// Define simple RideBooking schema & model
const rideSchema = new mongoose.Schema({
  phone: String,
  pickup: String,
  dropoff: String,
  status: { type: String, default: "pending" },
  createdAt: { type: Date, default: Date.now },
});
const RideBooking = mongoose.model("RideBooking", rideSchema);
```

---

3. Update the `/whatsapp` route to parse intent & save booking

Replace the OpenAI call with something like this:

```js
app.post("/whatsapp", async (req, res) => {
  const from = req.body.From; // WhatsApp sender number
  const msg = req.body.Body.toLowerCase();

  let reply = "";

  if (msg.includes("book ride") || msg.includes("ride from")) {
    // Naive parse example
    const match = msg.match(/ride from (.+) to (.+)/);
    if (match) {
const pickup = match[1];
      const dropoff = match[2];

      // Save booking to DB
      await RideBooking.create({ phone: from, pickup, dropoff });

      reply = `Thanks! Your ride from pickup to{dropoff} is booked. We will notify you soon.`;
    } else {
      reply = "To book a ride, please send: 'Ride from [pickup location] to [dropoff location]'.";
    }
  } else if (msg.includes("hello") || msg.includes("hi")) {
    reply = "Hello! You can book rides by saying 'Ride from [pickup] to [dropoff]'.";
  } else {
    reply = "Sorry, I didn't understand. Try 'Book ride from [pickup] to [dropoff]'.";
  }

  // Send reply
  await client.messages.create({
    from: process.env.TWILIO_WHATSAPP_NUMBER,
    to: from,
    body: reply,
  });

  res.sendStatus(200);
});
```

---

What this does:

- Detects when user wants to book a ride with simple keyword + regex  
- Saves ride bookings in MongoDB with their WhatsApp number  
- Replies with confirmation or help text  

---

You can extend this by:

- Parsing food orders, dispatch requests similarly  
- Using OpenAI to classify intents better before action  
- Sending ride status updates to users via WhatsApp  
- Connecting this with your full backend services  

---
---

- Sends user message to OpenAI with a prompt that requests intent classification + entity extraction  
- Parses the JSON response from AI  
- Handles each intent by saving bookings, replying appropriately  
- Fallbacks if AI response is malformed or unknown intent  

---

3. Next you can:

- Expand OrderFood and DispatchParcel handlers to save to MongoDB and confirm  
- Add onboarding flows like registration (collect user details)  
- Implement WhatsApp verification with OTP for user identity  
- Connect AI responses to trigger your backend workflows  

---

If you want, I can write the *WhatsApp onboarding + OTP verification code next* — just ask!1. Update your `/whatsapp` route to use OpenAI intent classification

Replace your current message handler with this:

```js
app.post("/whatsapp", async (req, res) => {
  const from = req.body.From;
  const msg = req.body.Body;

  // Prompt OpenAI to classify intent and extract entities
  const prompt = `
You are CityMap AI assistant. Identify the user's intent as one of: BookRide, OrderFood, DispatchParcel, Greeting, or Unknown.
If intent is BookRide, extract pickup and dropoff locations.
If OrderFood, extract restaurant name and food items.
If DispatchParcel, extract pickup, dropoff and parcel details.
Respond in JSON ONLY like this:

{
  "intent": "BookRide",
  "pickup": "location",
  "dropoff": "location"
}

User message: "${msg}"
`;

  try {
    const completion = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [{ role: "system", content: prompt }],
    });

    const responseText = completion.choices[0].message.content.trim();

    // Parse JSON response safely
    let data;
    try {
      data = JSON.parse(responseText);
    } catch {
let reply = "";

    switch (data.intent) 
      case "BookRide":
        if (data.pickup        data.dropoff) 
          await RideBooking.create(
            phone: from,
            pickup: data.pickup,
            dropoff: data.dropoff,
          );
          reply = `Your ride from{data.pickup} to ${data.dropoff} has been booked!`;
        } else {
          reply = "Please provide pickup and dropoff locations for your ride.";
        }
        break;

      case "OrderFood":
        // Handle food orders similarly here
        reply = "Food ordering is coming soon!";
        break;

      case "DispatchParcel":
        // Handle dispatch orders similarly here
        reply = "Parcel dispatch service coming soon!";
        break;

      case "Greeting":
        reply = "Hello! How can I assist you with CityMap today?";
        break;

      default:
        reply = "Sorry, I didn't understand that. You can book rides or order food.";
    }

    await client.messages.create({
      from: process.env.TWILIO_WHATSAPP_NUMBER,
      to: from,
      body: reply,
    });

    res.sendStatus(200);
  } catch (error) {
    console.error(error);
    res.sendStatus(500);
  }
});
```

---

2. What this does:
- Sends user message to OpenAI with a prompt that requests intent classification + entity extraction  
- Parses the JSON response from AI  
- Handles each intent by saving bookings, replying appropriately  
- Fallbacks if AI response is malformed or unknown intent  

---

3. Next you can:

- Expand OrderFood and DispatchParcel handlers to save to MongoDB and confirm  
- Add onboarding flows like registration (collect user details)  
- Implement WhatsApp verification with OTP for user identity  
- Connect AI responses to trigger your backend workflows  

---

---

1. Setup Twilio Verify

- In your Twilio console, create a Verify Service and get its SID (`TWILIO_VERIFY_SERVICE_SID`).

---

2. Add `.env` variables:

```
TWILIO_VERIFY_SERVICE_SID=your_verify_service_sid
```

---

3. Update your `whatsappBot.js`:

```js
// Add at top
const verifyServiceSid = process.env.TWILIO_VERIFY_SERVICE_SID;

// New endpoint to start verification
app.post("/start-verification", async (req, res) => {
  const phone = req.body.phone; // expects format: whatsapp:+123456789

  try {
    await client.verify.services(verifyServiceSid).verifications.create({
      to: phone,
      channel: "whatsapp",
    });
    res.json({ status: "pending", message: "OTP sent via WhatsApp" });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});

// New endpoint to check verification code
app.post("/check-verification", async (req, res) => {
  const phone = req.body.phone;
  const code = req.body.code;

  try {
    const verification_check = await client.verify.services(verifyServiceSid)
      .verificationChecks.create({
        to: phone,
        code: code,
      });

    if (verification_check.status === "approved") {
// Mark user as verified in your DB or create user profile here
      res.json({ verified: true, message: "Phone verified!" });
    } else {
      res.json({ verified: false, message: "Invalid code, try again." });
    }
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
});
```

---

4. How to use:

- Call `/start-verification` with `{ phone: "whatsapp:+123456789" }` to send OTP  
- User receives OTP in WhatsApp and sends it back  
- Call `/check-verification` with `{ phone: "whatsapp:+123456789", code: "123456" }` to verify  
- On success, register or log in user in your system  

---

5. Integrate with your WhatsApp bot `/whatsapp` route

You can prompt new users to verify their number before allowing bookings, e.g.:

```js
if (!userIsVerified(from)) {
  reply = "Welcome! Please verify your phone by sending 'VERIFY' to start.";
} else {
  // handle intents as before
}
```

---

- List users with phone & verification status  
- Show rides booked by users  
- Allow admin to update ride status  

---

1. Install Axios for API calls:

```bash
npm install axios
```

---

2. Create `Dashboard.jsx`

```jsx
import React, { useEffect, useState } from "react";
import axios from "axios";

export default function Dashboard() {
  const [users, setUsers] = useState([]);
  const [rides, setRides] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    async function fetchData() {
      try {
        const usersRes = await axios.get("/api/users");
        const ridesRes = await axios.get("/api/rides");
        setUsers(usersRes.data);
        setRides(ridesRes.data);
      } catch (err) {
        console.error(err);
      } finally {
        setLoading(false);
      }
    }
    fetchData();
  }, []);

  const updateRideStatus = async (rideId, status) => {
    try {
      await axios.put(`/api/rides/${rideId}`, { status });
      setRides((prev) =>
        prev.map((r) => (r._id === rideId ? { ...r, status } : r))
      );
    } catch (err) {
      console.error(err);
    }
  };

  if (loading) return <div>Loading dashboard...</div>;

return (
    <div>
      <h2>Users</h2>
      <table border="1" cellPadding="5">
        <thead>
          <tr>
            <th>Phone</th>
            <th>Verified</th>
          </tr>
        </thead>
        <tbody>
          {users.map((u) => (
            <tr key={u._id}>
              <td>{u.phone}</td>
              <td>{u.verified ? "Yes" : "No"}</td>
            </tr>
          ))}
        </tbody>
      </table>

      <h2>Rides</h2>
      <table border="1" cellPadding="5">
        <thead>
          <tr>
            <th>User Phone</th>
            <th>Pickup</th>
            <th>Dropoff</th>
            <th>Status</th>
            <th>Update Status</th>
          </tr>
        </thead>
        <tbody>
          {rides.map((r) => (
            <tr key={r._id}>
              <td>{r.phone}</td>
              <td>{r.pickup}</td>
              <td>{r.dropoff}</td>
              <td>{r.status}</td>
              <td>
                <select
                  value={r.status}
                  onChange={(e) => updateRideStatus(r._id, e.target.value)}
                >
                  <option value="pending">Pending</option>
                  <option value="accepted">Accepted</option>
                  <option value="in_progress">In Progress</option>

<option value="completed">Completed</option>
                  <option value="cancelled">Cancelled</option>
                </select>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

3. Backend API endpoints example (Express + MongoDB)

Add to your backend:

```js
// User model example
const userSchema = new mongoose.Schema({
  phone: String,
  verified: Boolean,
});
const User = mongoose.model("User", userSchema);

// GET /api/users
app.get("/api/users", async (req, res) => {
  const users = await User.find({});
  res.json(users);
});

// GET /api/rides
app.get("/api/rides", async (req, res) => {
  const rides = await RideBooking.find({});
  res.json(rides);
});

// PUT /api/rides/:id
app.put("/api/rides/:id", async (req, res) => {
  const { status } = req.body;
  const ride = await RideBooking.findByIdAndUpdate(
    req.params.id,
    { status },
    { new: true }
  );
  res.json(ride);
});
```

---

4. Usage

- Add `<Dashboard />` in your React app  
- Ensure API proxy or CORS is set up for `/api/*` calls to backend  

---

---

1. Install dependencies

If you don’t have it yet:

```bash
npm install axios
```

---

2. Create `AICopilot.jsx`

```jsx
import React, { useState, useEffect, useRef } from "react";
import axios from "axios";

export default function AICopilot() {
  const [messages, setMessages] = useState([
    { from: "bot", text: "Hi! I'm your CityMap AI co-pilot. How can I help you today?" },
  ]);
  const [input, setInput] = useState("");
  const messagesEndRef = useRef(null);

  const scrollToBottom = () => {
    messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
  };

  useEffect(scrollToBottom, [messages]);

  const sendMessage = async () => {
    if (!input.trim()) return;

    const userMsg = { from: "user", text: input };
    setMessages((prev) => [...prev, userMsg]);

    try {
      const res = await axios.post("/api/ai-chat", { message: input });
      const botReply = res.data.reply || "Sorry, I didn't understand that.";

      setMessages((prev) => [...prev, { from: "bot", text: botReply }]);
    } catch (err) {
      setMessages((prev) => [
        ...prev,
{ from: "bot", text: "Oops! Something went wrong." },
      ]);
    }

    setInput("");
  };

  const handleKeyPress = (e) => {
    if (e.key === "Enter") sendMessage();
  };

  return (
    <div style={{ maxWidth: 500, margin: "auto", padding: 20, border: "1px solid #ccc", borderRadius: 8 }}>
      <div
        style={{
          height: 400,
          overflowY: "auto",
          border: "1px solid #eee",
          padding: 10,
          marginBottom: 10,
          backgroundColor: "#f9f9f9",
        }}
      >
        {messages.map((m, i) => (
          <div
            key={i}
            style={{
              textAlign: m.from === "user" ? "right" : "left",
              margin: "8px 0",
            }}
          >
            <span
              style={{
                display: "inline-block",
                padding: "8px 12px",
                borderRadius: 20,
                backgroundColor: m.from === "user" ? "#007bff" : "#e5e5ea",
                color: m.from === "user" ? "#fff" : "#000",
                maxWidth: "80%",
              }}
            >
              {m.text}
            </span>
          </div>
        ))}
        <div ref={messagesEndRef} />
      </div>
      <input
        type="text"
        placeholder="Ask me anything..."
value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyPress={handleKeyPress}
        style={{ width: "100%", padding: 10, borderRadius: 4, border: "1px solid #ccc" }}
      />
      <button
        onClick={sendMessage}
        style={{ marginTop: 10, width: "100%", padding: 10, borderRadius: 4, backgroundColor: "#007bff", color: "white", border: "none" }}
      >
        Send
      </button>
    </div>
  );
}
```

---

3. Backend API endpoint `/api/ai-chat`

Add this to your backend (Express):

```js
app.use(express.json());

app.post("/api/ai-chat", async (req, res) => {
  const { message } = req.body;

  try {
    const completion = await openai.chat.completions.create({
      model: "gpt-4o-mini",
      messages: [
        {
          role: "system",
          content:
            "You are CityMap AI co-pilot assistant. Help user with rides, food orders, traffic, and more.",
        },
        { role: "user", content: message },
      ],
    });

    const reply = completion.choices[0].message.content;
    res.json({ reply });
  } catch (error) {
    console.error(error);
    res.status(500).json({ reply: "Sorry, I couldn't process that." });
  }
});
```

---

4. How to use

- Import and render `<AICopilot />` in your React app
- Make sure API calls to `/api/ai-chat` reach your backend (set proxy or CORS)  
- The AI co-pilot chat window will appear and let users ask anything  

---

- A backend that tracks ride status & location in real-time  
- Integration with a mapping API (like Google Maps) for traffic data  
- Your AI co-pilot to query this data and respond  

---

Step 1: Real-time ride tracking backend (using Socket.IO)

Install:

```bash
npm install socket.io
```

In your backend (`server.js` or similar):

```js
import { Server } from "socket.io";

const io = new Server(httpServer, {
  cors: { origin: "*" },
});

const rideLocations = {}; // store rideId -> { lat, lng, status }

io.on("connection", (socket) => {
  console.log("Client connected:", socket.id);

  socket.on("updateLocation", ({ rideId, lat, lng, status }) => {
    rideLocations[rideId] = { lat, lng, status };
    // Broadcast to all clients tracking this ride
    io.emit("rideLocationUpdate", { rideId, lat, lng, status });
  });

  socket.on("disconnect", () => {
    console.log("Client disconnected:", socket.id);
  });
});
```

---

Step 2: Frontend ride map with live updates

Use `socket.io-client` and Google Maps React components.

Install:

```bash
npm install socket.io-client @react-google-maps/api
```

Example `LiveRideMap.jsx`:

```jsx 
import React, { useEffect, useState } from "react";
import { io } from "socket.io-client";
import { GoogleMap, Marker, useJsApiLoader } from "@react-google-maps/api";

const socket = io("http://localhost:YOUR_BACKEND_PORT");

export default function LiveRideMap({ rideId }) {
  const [location, setLocation] = useState(null);

  useEffect(() => {
    socket.on("rideLocationUpdate", (data) => {
      if (data.rideId === rideId) {
        setLocation({ lat: data.lat, lng: data.lng });
      }
    });

    return () => {
      socket.off("rideLocationUpdate");
    };
  }, [rideId]);

  const { isLoaded } = useJsApiLoader({
    googleMapsApiKey: "YOUR_GOOGLE_MAPS_API_KEY",
  });

  if (!isLoaded) return <div>Loading map...</div>;

  return (
    <GoogleMap
      center={location || { lat: 0, lng: 0 }}
      zoom={15}
      mapContainerStyle={{ width: "100%", height: "400px" }}
    >
      {location && <Marker position={location} />}
    </GoogleMap>
  );
}
```

---

Step 3: AI co-pilot queries ride status & traffic info

Update your AI co-pilot backend to:

- Access current ride location & status from `rideLocations` or DB  
- Use Google Maps Traffic API or Directions API to fetch traffic data  
- Incorporate this info into the AI response prompt  

Example snippet:

```js 
// Inside /api/ai-chat handler

const rideId = getUserCurrentRideId(userPhone); // your function to get current ride

const rideInfo = rideLocations[rideId] || {};
const trafficInfo = await getTrafficInfo(rideInfo.lat, rideInfo.lng); // your function

const prompt = `
You are CityMap AI assistant. Current ride status: rideInfo.status || "none". Location:{rideInfo.lat || "N/A"}, rideInfo.lng || "N/A". Traffic update:{trafficInfo || "No data"}.
Answer user question: ${message}
`;

const completion = await openai.chat.completions.create({
  model: "gpt-4o-mini",
  messages: [{ role: "system", content: prompt }, { role: "user", content: message }],
});
```

---

Step 4: Tie it all together

- When a driver updates location via your app, emit `"updateLocation"` socket event  
- User’s frontend listens to `"rideLocationUpdate"` to update the map live  
- User chats with AI co-pilot that has real-time ride + traffic context  
- AI answers with live updates on ride ETA, delays, or reroutes  

---

- Track driver location in real time  
- Fetch live traffic updates between pickup and dropoff  
- Feed this to the AI or display it in the dashboard/app  

---

✅ PART 1: DRIVER LOCATION UPDATER (Mobile/Web)

Use `navigator.geolocation` in a simple React app to emit location:

*`DriverTracker.jsx`*

```jsx
import React, { useEffect, useState } from "react";
import { io } from "socket.io-client";

const socket = io("http://localhost:YOUR_BACKEND_PORT"); // or your live domain

export default function DriverTracker({ rideId }) {
  const [status, setStatus] = useState("on_trip");

  useEffect(() => {
    const watchId = navigator.geolocation.watchPosition(
      (position) => {
        const { latitude: lat, longitude: lng } = position.coords;
        socket.emit("updateLocation", { rideId, lat, lng, status });
      },
      (err) => console.error("Location error", err),
      { enableHighAccuracy: true, maximumAge: 5000, timeout: 10000 }
    );

    return () => navigator.geolocation.clearWatch(watchId);
  }, [rideId, status]);

  return (
    <div>
      <p>Tracking ride: {rideId}</p>
      <select value={status} onChange={(e) => setStatus(e.target.value)}> 
<option value="on_trip">On Trip</option>
        <option value="paused">Paused</option>
        <option value="completed">Completed</option>
      </select>
    </div>
  );
}
```

---

✅ PART 2: GOOGLE TRAFFIC / DURATION API

Use Google’s *Directions API* to get ETA and traffic conditions.

Install Axios if not already:
```bash
npm install axios
```

Backend helper function: `getTrafficInfo`

```js
import axios from "axios";

export async function getTrafficInfo(pickup, dropoff) {
  const apiKey = process.env.GOOGLE_MAPS_API_KEY;

  const url = `https://maps.googleapis.com/maps/api/directions/json?origin=pickup   destination={dropoff}&departure_time=now&key=${apiKey}`;

  try {
    const res = await axios.get(url);
    const route = res.data.routes[0];
    const leg = route.legs[0];

    return {
      eta: leg.duration_in_traffic.text,
      distance: leg.distance.text,
      summary: route.summary,
    };
  } catch (e) {
    console.error("Traffic API error", e);
    return null;
  }
}
```

---

✅ PART 3: AI Copilot Uses Traffic Data

Update `/api/ai-chat`:

```js
import { getTrafficInfo } from "./trafficHelper.js";

app.post("/api/ai-chat", async (req, res) => {
  const { message } = req.body;
  const phone = req.body.phone; // user number 
const ride = await RideBooking.findOne( phone, status: "in_progress" );

  let traffic = "No active ride.";
  if (ride) 
    const trafficInfo = await getTrafficInfo(ride.pickup, ride.dropoff);
    if (trafficInfo)
      traffic = `Distance:{trafficInfo.distance}, ETA: trafficInfo.eta, via{trafficInfo.summary}`;
  }

  const prompt = `
You are CityMap AI assistant. User ride info: traffic
User:{message}
`;

  const completion = await openai.chat.completions.create({
    model: "gpt-4o-mini",
    messages: [{ role: "system", content: prompt }],
  });

  const reply = completion.choices[0].message.content;
  res.json({ reply });
});
```

---

Result:

- Driver’s location is updated live to server  
- AI copilot can tell the user:  
  “Your driver is 12 mins away via 3rd Mainland Bridge. Expect some traffic near Yaba.”  
- Traffic ETA adjusts in real-time  

---

---

✅ 1. Update MongoDB Models

Add two new schemas: *FoodOrder* and *DispatchOrder*

```js
// models/FoodOrder.js
const foodSchema = new mongoose.Schema({
  phone: String,
  restaurant: String,
  items: [String],
  status: { type: String, default: "pending" },
  createdAt: { type: Date, default: Date.now },
});
export const FoodOrder = mongoose.model("FoodOrder", foodSchema);

// models/DispatchOrder.js
const dispatchSchema = new mongoose.Schema({
  phone: String,
  pickup: String,
  dropoff: String,
  parcel: String,
  status: { type: String, default: "pending" },
  createdAt: { type: Date, default: Date.now },
});
export const DispatchOrder = mongoose.model("DispatchOrder", dispatchSchema);
```

---

✅ 2. AI Intent Parsing for Orders & Dispatches

Update `/api/ai-chat` backend logic to handle all 3:

```js
const prompt = `
You are CityMap AI. Classify user intent as: BookRide, OrderFood, DispatchParcel, or General.
Return a JSON like:
{
  "intent": "OrderFood",
  "restaurant": "Chicken Republic",
  "items": ["Jollof rice", "Chicken"]
}
Or:
{
  "intent": "DispatchParcel",
  "pickup": "Lekki",
  "dropoff": "Ikeja",
  "parcel": "Documents"
} 
User: "message"
`;

const completion = await openai.chat.completions.create(
  model: "gpt-4o-mini",
  messages: [ role: "system", content: prompt ],
);
const parsed = JSON.parse(completion.choices[0].message.content.trim());
“`

—

✅ 3. Handle Each Intent

“`js
let reply = "";

switch (parsed.intent) 
  case "BookRide":
    // existing ride booking logic
    break;

  case "OrderFood":
    if (parsed.restaurant        parsed.items?.length) 
      await FoodOrder.create(
        phone,
        restaurant: parsed.restaurant,
        items: parsed.items,
      );
      reply = `Food order placed at{parsed.restaurant} for: parsed.items.join(", ")`;
     else 
      reply = "Please provide restaurant name and items.";
    
    break;

  case "DispatchParcel":
    if (parsed.pickup        parsed.dropoff        parsed.parcel) 
      await DispatchOrder.create(
        phone,
        pickup: parsed.pickup,
        dropoff: parsed.dropoff,
        parcel: parsed.parcel,
      );
      reply = `Parcel pickup from{parsed.pickup} to parsed.dropoff for '{parsed.parcel}' scheduled.`;
    } else {
      reply = "Please include pickup, dropoff and parcel description.";
    }
    break;

  default: 
reply = "Hi! I can help you book rides, order food, or dispatch parcels. Try something like 'Order pizza from Domino’s'.";
}
```

---

✅ 4. Show on React Dashboard

In `Dashboard.jsx`, add tabs or sections for:

- *Food Orders* → GET `/api/food-orders`
- *Dispatch Orders* → GET `/api/dispatch-orders`

---

✅ 5. Optional AI Copilot Enhancements

Let AI copilot suggest:

- Nearby restaurants based on pickup area  
- Pre-fill dispatch forms after parcel conversations  
- Notify users via WhatsApp when orders are confirmed or delivered  

---

---

✅ 1. *CityEats Order Form*

*`CityEats.jsx`*

```jsx
import React, { useState } from "react";
import axios from "axios";

export default function CityEats() {
  const [phone, setPhone] = useState("");
  const [restaurant, setRestaurant] = useState("");
  const [items, setItems] = useState("");
  const [message, setMessage] = useState("");

  const placeOrder = async () => {
    try {
      const res = await axios.post("/api/food-orders", {
        phone,
        restaurant,
        items: items.split(",").map(i => i.trim()),
      });
      setMessage("Food order placed successfully!");
    } catch (err) {
      console.error(err);
      setMessage("Failed to place order.");
    }
  };

  return (
    <div>
      <h3>🍽️ CityEats - Order Food</h3>
      <input placeholder="Your phone" value={phone} onChange={e => setPhone(e.target.value)} /><br />
      <input placeholder="Restaurant" value={restaurant} onChange={e => setRestaurant(e.target.value)} /><br />
      <input placeholder="Food items (comma separated)" value={items} onChange={e => setItems(e.target.value)} /><br /> 
<button onClick={placeOrder}>Place Order</button>
      <p>{message}</p>
    </div>
  );
}
```

---

✅ 2. *CityDispatch Parcel Form*

*`CityDispatch.jsx`*

```jsx
import React, { useState } from "react";
import axios from "axios";

export default function CityDispatch() {
  const [phone, setPhone] = useState("");
  const [pickup, setPickup] = useState("");
  const [dropoff, setDropoff] = useState("");
  const [parcel, setParcel] = useState("");
  const [message, setMessage] = useState("");

  const sendParcel = async () => {
    try {
      await axios.post("/api/dispatch-orders", {
        phone,
        pickup,
        dropoff,
        parcel,
      });
      setMessage("Dispatch order placed successfully!");
    } catch (err) {
      console.error(err);
      setMessage("Failed to place dispatch.");
    }
  };

  return (
    <div>
      <h3>📦 CityDispatch - Send Parcel</h3>
      <input placeholder="Your phone" value={phone} onChange={e => setPhone(e.target.value)} /><br />
      <input placeholder="Pickup location" value={pickup} onChange={e => setPickup(e.target.value)} /><br />
      <input placeholder="Dropoff location" value={dropoff} onChange={e => setDropoff(e.target.value)} /><br /> 
<input placeholder="Parcel details" value={parcel} onChange={e => setParcel(e.target.value)} /><br />
      <button onClick={sendParcel}>Send Parcel</button>
      <p>{message}</p>
    </div>
  );
}
```

---

✅ 3. Backend API Endpoints

Add in Express backend:

```js
app.post("/api/food-orders", async (req, res) => {
  const { phone, restaurant, items } = req.body;
  const order = await FoodOrder.create({ phone, restaurant, items });
  res.json(order);
});

app.post("/api/dispatch-orders", async (req, res) => {
  const { phone, pickup, dropoff, parcel } = req.body;
  const order = await DispatchOrder.create({ phone, pickup, dropoff, parcel });
  res.json(order);
});
```

---

✅ 4. Use in Main App

In your main React app:

```jsx
import CityEats from './CityEats';
import CityDispatch from './CityDispatch';

function App() {
  return (
    <div>
      <CityEats />
      <hr />
      <CityDispatch />
    </div>
  );
}
```

---

---

✅ 1. Prerequisites

Make sure you have:

- A verified *Twilio WhatsApp number*  
- Your *Twilio Account SID*, *Auth Token*, and *WhatsApp Sender Number*  
- Added in `.env`:

```
TWILIO_SID=your_account_sid
TWILIO_TOKEN=your_auth_token
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
```

---

✅ 2. Setup Twilio in your backend

*`twilioClient.js`*

```js
import twilio from "twilio";

const client = twilio(process.env.TWILIO_SID, process.env.TWILIO_TOKEN);

export function sendWhatsAppMessage(to, message) {
  return client.messages.create({
    from: process.env.TWILIO_WHATSAPP_NUMBER,
    to: `whatsapp:to`,
    body: message,
  );
“`

—

✅ 3. Send WhatsApp When Food Order Is Placed

Update `/api/food-orders`:

“`js
import  sendWhatsAppMessage  from "./twilioClient.js";

app.post("/api/food-orders", async (req, res) => 
  const  phone, restaurant, items  = req.body;
  const order = await FoodOrder.create( phone, restaurant, items );

  const text = `🍽️ CityMap - Food Order Confirmed!:{restaurant}\nItems: ${items.join(", ")}\nWe’ll notify you when it’s on the way.`;

  await sendWhatsAppMessage(phone, text);

  res.json(order);
});
```

--- 
✅ 4. Send WhatsApp When Dispatch Order Is Placed

Update `/api/dispatch-orders`:

“`js
app.post("/api/dispatch-orders", async (req, res) => 
  const  phone, pickup, dropoff, parcel  = req.body;
  const order = await DispatchOrder.create( phone, pickup, dropoff, parcel );

  const text = `📦 CityMap - Dispatch Confirmed!:{parcel}\nPickup: pickup:{dropoff}\nWe'll keep you updated on the trip.`;

  await sendWhatsAppMessage(phone, text);

  res.json(order);
});
```

---

✅ 5. Bonus (Optional)

Later, you can send follow-ups like:

- “Your food is on the way 🚗 ETA: 10 mins”  
- “Your parcel has been delivered ✅”  

Just call `sendWhatsAppMessage()` from any order status update logic.

---

---

✅ 1. Update Order Models with Status

Make sure both models already have a `status` field (default: `pending`).

If not, update like this:

```js
status: { type: String, default: "pending" } // e.g. 'preparing', 'on_the_way', 'delivered'
```

---

✅ 2. Add Backend Endpoints to Update Status

Add this to Express:

```js
// Update food order status
app.put("/api/food-orders/:id/status", async (req, res) => {
  const { status } = req.body;
  const order = await FoodOrder.findByIdAndUpdate(req.params.id, { status }, { new: true });

  if (order) {
    let msg = "";
    switch (status) {
      case "preparing":
        msg = `🍽️ Your food is being prepared at ${order.restaurant}.`;
        break;
      case "on_the_way":
        msg = `🚗 Your food is on the way!`;
        break;
      case "delivered":
        msg = `✅ Your food order has been delivered. Enjoy your meal!`;
        break;
    }
    await sendWhatsAppMessage(order.phone, msg);
  }

  res.json(order);
});

// Update dispatch status
app.put("/api/dispatch-orders/:id/status", async (req, res) => {
  const { status } = req.body; 
const order = await DispatchOrder.findByIdAndUpdate(req.params.id,  status ,  new: true );

  if (order) 
    let msg = "";
    switch (status) 
      case "pickup_confirmed":
        msg = `📦 Your parcel has been picked up from{order.pickup}.`;
        break;
      case "on_the_way":
        msg = `🚚 Your parcel is en route to order.dropoff.`;
        break;
      case "delivered":
        msg = `✅ Your parcel has been delivered to{order.dropoff}.`;
        break;
    }
    await sendWhatsAppMessage(order.phone, msg);
  }

  res.json(order);
});
```

---

✅ 3. Update Status from Dashboard (Optional)

In your admin dashboard or delivery interface, send a `PUT` request to:

- `/api/food-orders/:id/status`  
- `/api/dispatch-orders/:id/status`  

With a body like:

```json
{ "status": "on_the_way" }
```

—

---

✅ 1. Admin Dashboard for Food Orders

*`FoodOrdersAdmin.jsx`*

```jsx
import React, { useEffect, useState } from "react";
import axios from "axios";

export default function FoodOrdersAdmin() {
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    axios.get("/api/food-orders").then((res) => setOrders(res.data));
  }, []);

  const updateStatus = async (id, status) => {
    await axios.put(`/api/food-orders/${id}/status`, { status });
    setOrders((prev) =>
      prev.map((o) => (o._id === id ? { ...o, status } : o))
    );
  };

  return (
    <div>
      <h3>🍽️ Food Orders (Admin)</h3>
      {orders.map((o) => (
        <div key={o._id} style={{ border: "1px solid #ccc", margin: 10, padding: 10 }}>
          <p><b>Restaurant:</b> {o.restaurant}</p>
          <p><b>Items:</b> {o.items.join(", ")}</p>
          <p><b>Status:</b> {o.status}</p>
          <button onClick={() => updateStatus(o._id, "preparing")}>Preparing</button>
          <button onClick={() => updateStatus(o._id, "on_the_way")}>On the Way</button>
          <button onClick={() => updateStatus(o._id, "delivered")}>Delivered</button>
        </div> 
“`

—

✅ 2. Admin Dashboard for Dispatch Orders

*`DispatchOrdersAdmin.jsx`*

“`jsx
import React,  useEffect, useState  from "react";
import axios from "axios";

export default function DispatchOrdersAdmin() 
  const [orders, setOrders] = useState([]);

  useEffect(() => 
    axios.get("/api/dispatch-orders").then((res) => setOrders(res.data));
  , []);

  const updateStatus = async (id, status) => 
    await axios.put(`/api/dispatch-orders/{id}/status`, { status });
    setOrders((prev) =>
      prev.map((o) => (o._id === id ? { ...o, status } : o))
    );
  };

  return (
    <div>
      <h3>📦 Dispatch Orders (Admin)</h3>
      {orders.map((o) => (
        <div key={o._id} style={{ border: "1px solid #ccc", margin: 10, padding: 10 }}>
          <p><b>Parcel:</b> {o.parcel}</p>
          <p><b>Pickup:</b> {o.pickup}</p>
          <p><b>Dropoff:</b> {o.dropoff}</p>
          <p><b>Status:</b> {o.status}</p>
          <button onClick={() => updateStatus(o._id, "pickup_confirmed")}>Picked Up</button>
          <button onClick={() => updateStatus(o._id, "on_the_way")}>On the Way</button>
          <button onClick={() => updateStatus(o._id, "delivered")}>Delivered</button>
        </div>
      ))}
    </div>
  );
}
```

---

✅ 3. Add Admin Routes on Backend 
Add two GET endpoints if not yet added:

```js
app.get("/api/food-orders", async (req, res) => {
  const orders = await FoodOrder.find().sort({ createdAt: -1 });
  res.json(orders);
});

app.get("/api/dispatch-orders", async (req, res) => {
  const orders = await DispatchOrder.find().sort({ createdAt: -1 });
  res.json(orders);
});
```

---

---

1. *Backend: Basic Auth Middleware*

Add this in your Express server to protect admin routes:

```js
const basicAuth = (req, res, next) => {
  const auth = req.headers.authorization;

  if (!auth) {
    res.setHeader('WWW-Authenticate', 'Basic');
    return res.status(401).send('Authentication required.');
  }

  const [user, pass] = Buffer.from(auth.split(' ')[1], 'base64').toString().split(':');

  if (user === process.env.ADMIN_USER && pass === process.env.ADMIN_PASS) {
    return next();
  }

  res.setHeader('WWW-Authenticate', 'Basic');
  return res.status(401).send('Authentication failed.');
};

// Protect admin API routes
app.use('/api/admin', basicAuth);

// Example protected route
app.get('/api/admin/food-orders', async (req, res) => {
  const orders = await FoodOrder.find().sort({ createdAt: -1 });
  res.json(orders);
});

// Similarly protect other admin APIs with /api/admin prefix
```

Add `.env` variables:

```
ADMIN_USER=admin
ADMIN_PASS=yourStrongPassword
```

---

2. *Backend: Update Routes Prefix*

Change admin routes to `/api/admin/...`

Example:

```js
app.get('/api/admin/food-orders', ...);
app.get('/api/admin/dispatch-orders', ...); 
app.put('/api/admin/food-orders/:id/status', ...);
app.put('/api/admin/dispatch-orders/:id/status', ...);
```

---

3. *Frontend: Combined Admin Dashboard*

*`AdminDashboard.jsx`*

```jsx
import React, { useState } from "react";
import FoodOrdersAdmin from "./FoodOrdersAdmin";
import DispatchOrdersAdmin from "./DispatchOrdersAdmin";

export default function AdminDashboard() {
  const [tab, setTab] = useState("food");

  return (
    <div>
      <h2>CityMap Admin Panel</h2>
      <button onClick={() => setTab("food")}>Food Orders</button>
      <button onClick={() => setTab("dispatch")}>Dispatch Orders</button>

      {tab === "food" && <FoodOrdersAdmin />}
      {tab === "dispatch" && <DispatchOrdersAdmin />}
    </div>
  );
}
```

---

4. *Frontend: Update API URLs*

In your admin components, update all API calls to prefix `/api/admin` and include basic auth headers.

Example (in `FoodOrdersAdmin.jsx`):

```js
const authHeader = {
  headers: {
    Authorization: "Basic " + btoa("admin:yourStrongPassword")
  }
};

useEffect(() => {
  axios.get("/api/admin/food-orders", authHeader).then((res) => setOrders(res.data));
}, []);

const updateStatus = async (id, status) => {
  await axios.put(`/api/admin/food-orders/${id}/status`, { status }, authHeader);
  // update state...
};
``` 
---

Summary:

- Server API protected with *Basic Auth*  
- Admin React UI lets you toggle between Food & Dispatch orders  
- All requests send *Basic Auth headers*  

---


---

✅ 1. *Backend: Setup JWT Auth (Express)*

*Install:*

```bash
npm install jsonwebtoken bcryptjs
```

---

*Add admin model (optional if only 1 admin):*

```js
const AdminSchema = new mongoose.Schema({
  username: String,
  passwordHash: String,
});

export const Admin = mongoose.model("Admin", AdminSchema);
```

---

*Register Admin (once):*

```js
import bcrypt from "bcryptjs";

app.post("/api/admin/register", async (req, res) => {
  const { username, password } = req.body;
  const passwordHash = await bcrypt.hash(password, 10);
  const admin = await Admin.create({ username, passwordHash });
  res.json({ message: "Admin created", admin });
});
```

---

*Login Route:*

```js
import jwt from "jsonwebtoken";

app.post("/api/admin/login", async (req, res) => {
  const { username, password } = req.body;
  const admin = await Admin.findOne({ username });
  if (!admin) return res.status(401).send("Invalid");

  const match = await bcrypt.compare(password, admin.passwordHash);
  if (!match) return res.status(401).send("Invalid");

  const token = jwt.sign({ username }, process.env.JWT_SECRET, { expiresIn: "2h" });
  res.json({ token });
});
```

---

*Auth Middleware:*

```js 
Update all Axios calls:

```js
const token = localStorage.getItem("token");
const authHeader = { headers: { Authorization: `Bearer ${token}` } };

axios.get("/api/admin/food-orders", authHeader)
```

---

✅ 4. *Main App Entry (Switch Between Login & Admin)*

```jsx
import React, { useState, useEffect } from "react";
import AdminDashboard from "./AdminDashboard";
import Login from "./Login";

export default function App() {
  const [auth, setAuth] = useState(false);

  useEffect(() => {
    const token = localStorage.getItem("token");
    setAuth(!!token);
  }, []);

  return auth ? <AdminDashboard /> : <Login onLogin={() => setAuth(true)} />;
}
```

---

---

✅ 1. Token Strategy

We’ll use:

- *Access Token* — short-lived (e.g. 15 min), stored in memory
- *Refresh Token* — long-lived (e.g. 7 days), stored in `localStorage`

---

✅ 2. Backend: Issue Both Tokens on Login

Update `/api/admin/login`:

```js
const jwt = require("jsonwebtoken");

app.post("/api/admin/login", async (req, res) => {
  const { username, password } = req.body;
  const admin = await Admin.findOne({ username });
  if (!admin) return res.status(401).send("Invalid");

  const match = await bcrypt.compare(password, admin.passwordHash);
  if (!match) return res.status(401).send("Invalid");

  const accessToken = jwt.sign({ username }, process.env.JWT_SECRET, { expiresIn: "15m" });
  const refreshToken = jwt.sign({ username }, process.env.JWT_REFRESH_SECRET, { expiresIn: "7d" });

  res.json({ accessToken, refreshToken });
});
```

---

✅ 3. Add Refresh Token Endpoint

```js
app.post("/api/admin/refresh", (req, res) => {
  const { refreshToken } = req.body;
  if (!refreshToken) return res.status(401).send("No token");

  try {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET); 
const newAccessToken = jwt.sign({ username: decoded.username }, process.env.JWT_SECRET, { expiresIn: "15m" });
    res.json({ accessToken: newAccessToken });
  } catch {
    res.status(403).send("Invalid token");
  }
});
```

---

✅ 4. Frontend Auth Storage

Store tokens like this:

```js
localStorage.setItem("refresh", res.data.refreshToken);
sessionStorage.setItem("access", res.data.accessToken); // clear on browser close
```

---

✅ 5. Axios Auto Refresh Logic

Create `authHeader.js`:

```js
import axios from "axios";

export const getAuthHeader = async () => {
  let token = sessionStorage.getItem("access");

  try {
    // Try test request with current access token
    await axios.get("/api/admin/ping", {
      headers: { Authorization: `Bearer token` ,
    );
   catch (err) 
    // If access token expired, refresh it
    const refresh = localStorage.getItem("refresh");
    const res = await axios.post("/api/admin/refresh",  refreshToken: refresh );
    token = res.data.accessToken;
    sessionStorage.setItem("access", token);
  

  return 
    headers:  Authorization: `Bearer{token}` },
  };
};
```

Use like this in admin components:

```js
import { getAuthHeader } from "./authHeader";

const fetchOrders = async () => {
  const config = await getAuthHeader(); 
const res = await axios.get("/api/admin/food-orders", config);
  setOrders(res.data);
};
```

---

✅ 6. Logout

*Logout button:*

```jsx
<button onClick={() => {
  sessionStorage.clear();
  localStorage.removeItem("refresh");
  window.location.reload();
}}>Logout</button>
```

---

✅ 7. Backend Environment Variables

In `.env`:

```
JWT_SECRET=shortSecretKey
JWT_REFRESH_SECRET=longerRefreshSecret
```

---

✅ Now your admin login is *secure*, supports *logout*, and auto-refreshes tokens when expired. 

---

✅ 1. Extend Admin Model with Roles

Update your admin schema:

```js
const AdminSchema = new mongoose.Schema({
  username: String,
  passwordHash: String,
  role: { type: String, enum: ["admin", "dispatch", "support"], default: "admin" },
});
```

Example roles:

- *admin* — full access  
- *dispatch* — only see & update delivery orders  
- *support* — read-only for orders

---

✅ 2. Include Role in JWT

Update login token creation:

```js
const accessToken = jwt.sign({ username, role: admin.role }, process.env.JWT_SECRET, { expiresIn: "15m" });
const refreshToken = jwt.sign({ username, role: admin.role }, process.env.JWT_REFRESH_SECRET, { expiresIn: "7d" });
```

---

✅ 3. Middleware to Check Role

```js
const allowRoles = (...roles) => {
  return (req, res, next) => {
    const token = req.headers.authorization?.split(" ")[1];
    if (!token) return res.status(403).send("No token");

    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      if (!roles.includes(decoded.role)) return res.status(403).send("Forbidden");
      req.admin = decoded;
      next();
    } catch { 
res.status(403).send("Invalid token");
    }
  };
};
```

Use it like this:

```js
app.get("/api/admin/dispatch-orders", allowRoles("admin", "dispatch"), async (req, res) => {
  const orders = await DispatchOrder.find().sort({ createdAt: -1 });
  res.json(orders);
});
```

---

✅ 4. Frontend: Conditional Rendering by Role

In the dashboard, read the role from JWT:

```js
import jwtDecode from "jwt-decode";

const token = sessionStorage.getItem("access");
const { role } = jwtDecode(token);

return (
  <div>
    <h3>Welcome {role.toUpperCase()}</h3>
    {role !== "dispatch" && <FoodOrdersAdmin />}
    {(role === "admin" || role === "dispatch") && <DispatchOrdersAdmin />}
  </div>
);
```

---

✅ 5. Mobile-Responsive Admin UI

In your React CSS or use Tailwind/Bootstrap:

```css
/* Add media queries */
@media (max-width: 600px) {
  div {
    font-size: 14px;
    padding: 10px;
  }
  input, button {
    width: 100%;
    margin-bottom: 8px;
  }
}
```

Or wrap your UI in a `container` and use *flexbox/column layout* for smaller screens.

Also add `viewport` in `index.html`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0">
```

---

✅ Summary

✅ Roles: admin, dispatch, support  
✅ Role-based access to endpoints  
✅ JWT includes role 
✅ Responsive layout for phones  
✅ Conditional rendering in React

—

---

✅ 1. Convert React App to PWA

*Step 1:* Install PWA support

If using *Create React App*, it’s already built-in — just enable it:

In `src/index.js`, change:

```js
import * as serviceWorkerRegistration from './serviceWorkerRegistration';

serviceWorkerRegistration.register();
```

(If `serviceWorkerRegistration.js` doesn’t exist, run `npm run eject` or use a template like Vite + Workbox.)

---

*Step 2:* Add `manifest.json` in `public/`

```json
{
  "short_name": "CityMapAdmin",
  "name": "CityMap Admin Panel",
  "icons": [
    {
      "src": "/icon-192.png",
      "type": "image/png",
      "sizes": "192x192"
    },
    {
      "src": "/icon-512.png",
      "type": "image/png",
      "sizes": "512x512"
    }
  ],
  "start_url": ".",
  "display": "standalone",
  "theme_color": "#000000",
  "background_color": "#ffffff"
}
```

Then add to `index.html`:

```html
<link rel="manifest" href="%PUBLIC_URL%/manifest.json" />
<meta name="theme-color" content="#000000" />
```

Add icons (`icon-192.png`, `icon-512.png`) in `public/`.

---

✅ 2. Role-Based Views

*Inside `App.jsx`:*

```jsx
import jwtDecode from "jwt-decode"; 
import AdminDashboard from "./AdminDashboard";
import DispatchOrdersAdmin from "./DispatchOrdersAdmin";
import Login from "./Login";

export default function App() {
  const token = sessionStorage.getItem("access");

  if (!token) return <Login onLogin={() => window.location.reload()} />;

  const { role } = jwtDecode(token);

  return (
    <div>
      <h2>Welcome, {role.toUpperCase()}</h2>
      {role === "admin" && <AdminDashboard />}
      {role === "dispatch" && <DispatchOrdersAdmin />}
      {role === "support" && <p>Support view (read-only coming soon)</p>}
    </div>
  );
}
```

---

✅ 3. Mobile-Friendly UI

Wrap layouts in responsive containers or use TailwindCSS/Bootstrap. Add this to `index.html`:

```html
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
```

Now the app:

- Installs like a native app on phones
- Auto-refreshes tokens
- Shows different UI by role

—

Here’s how to *deploy your CityMap Admin PWA* using either *GitHub Pages* or *Vercel* — both free & easy.

---

✅ Option 1: Deploy to *GitHub Pages*

1. Add Homepage in `package.json`

```json
"homepage": "https://your-username.github.io/your-repo-name"
```

2. Install gh-pages

```bash
npm install gh-pages --save-dev
```

3. Add Scripts

```json
"scripts": {
  "predeploy": "npm run build",
  "deploy": "gh-pages -d build"
}
```

4. Deploy

```bash
npm run deploy
```

Done! It will be live at the URL in `homepage`.

---

✅ Option 2: Deploy to *Vercel* (Recommended for JWT + backend)

1. Push Your Fullstack Code to GitHub

Include:

- `/client` → React frontend  
- `/server` → Express backend  

2. Create Vercel Account & Import Project

- Go to *vercel.com*, log in with GitHub  
- Import your repo  
- Set root directory for frontend (`/client`)  
- Set *environment variables* for your backend:
  - `JWT_SECRET`
  - `JWT_REFRESH_SECRET`
  - `MONGO_URI`
  - `TWILIO_*`, etc.

3. Set API Directory (Optional)

If using Vercel functions for backend, place Express routes in `/api`.  
Otherwise, host backend separately on *Render.com*, *Railway*, etc.

---

✅ Bonus: Enable HTTPS and Auto-Updates

Both GitHub Pages and Vercel auto-enable HTTPS and update on push.

---

---

🚀 How It Works:

- *GitHub*: Stores all your source code (`client/`, `server/`, etc.)
- *Vercel*: Deploys your frontend directly from GitHub every time you push
- *Backend*: Can be hosted on Vercel Functions or another service (like Render or Railway)

---

✅ Step-by-Step Setup

1. *Organize Your Project*

```plaintext
citymap-platform/
│
├── client/         ← React PWA (CityMap Admin)
│   ├── public/
│   ├── src/
│   └── package.json
│
├── server/         ← Node.js + Express backend
│   ├── routes/
│   ├── models/
│   └── index.js
│
└── README.md
```

---

2. *Push to GitHub*

- Create a new GitHub repo (e.g. `CityMap`)
- Push the whole folder: `git push origin main`

---

3. *Deploy Frontend (client) to Vercel*

- Go to [vercel.com](https://vercel.com) → click *"Add Project"*
- Import your GitHub repo
- Set:
  - *Framework*: React
  - *Root directory*: `client`
- Click *Deploy*

✅ Your React PWA (Admin Panel) is now live with HTTPS

---

4. *Deploy Backend (server)*

*Option A:* Use [Render.com](https://render.com)

- New Web Service → connect GitHub → point to `/server`
- Set environment variables:
  - `JWT_SECRET`, `JWT_REFRESH_SECRET`
  - `MONGO_URI`, etc.
- Done ✅

*Option B:* Use Railway.app or deploy locally

---

5. *Frontend to Backend Connection*

In your React code (`client/src`):

```js
const API_BASE = "https://your-backend-url.com/api";

// Example:
axios.get(`${API_BASE}/admin/food-orders`, authHeader)
```

---

---

📁 Project Structure

```
citymap-platform/
├── client/            ← React PWA (admin panel)
│   ├── public/
│   │   └── manifest.json
│   ├── src/
│   │   ├── App.jsx
│   │   ├── index.js
│   │   ├── Login.jsx
│   │   ├── AdminDashboard.jsx
│   │   ├── DispatchOrdersAdmin.jsx
│   │   ├── FoodOrdersAdmin.jsx
│   │   └── authHeader.js
│   └── package.json
│
├── server/            ← Express + MongoDB backend
│   ├── models/
│   │   ├── Admin.js
│   │   ├── FoodOrder.js
│   │   └── DispatchOrder.js
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── foodRoutes.js
│   │   └── dispatchRoutes.js
│   ├── middleware/
│   │   └── auth.js
│   ├── .env
│   └── index.js
│
└── README.md
```

---

✅ CLIENT (React PWA)

Install with:

```bash
cd client
npm install
npm start
```

Example: `client/src/App.jsx`

```jsx
import React from "react";
import Login from "./Login";
import AdminDashboard from "./AdminDashboard";
import DispatchOrdersAdmin from "./DispatchOrdersAdmin";
import jwtDecode from "jwt-decode";

export default function App() {
  const token = sessionStorage.getItem("access");
  if (!token) return <Login onLogin={() => window.location.reload()} />;
const { role } = jwtDecode(token);

  return (
    <div>
      <h3>Welcome {role}</h3>
      {role === "admin" && <AdminDashboard />}
      {role === "dispatch" && <DispatchOrdersAdmin />}
      {role === "support" && <p>Read-only view coming soon</p>}
    </div>
  );
}
```

---

✅ SERVER (Express + MongoDB)

Install with:

```bash
cd server
npm install
node index.js
```

Example: `server/index.js`

```js
import express from "express";
import mongoose from "mongoose";
import dotenv from "dotenv";
import cors from "cors";

import authRoutes from "./routes/authRoutes.js";
import foodRoutes from "./routes/foodRoutes.js";
import dispatchRoutes from "./routes/dispatchRoutes.js";

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

app.use("/api/admin", authRoutes);
app.use("/api/admin", foodRoutes);
app.use("/api/admin", dispatchRoutes);

mongoose.connect(process.env.MONGO_URI).then(() => {
  app.listen(4000, () => console.log("API running on port 4000"));
});
```

---

✅ NEXT STEP

Let me know and I’ll generate:

- `authRoutes.js` (JWT login, refresh, logout)
- `authHeader.js` (Axios token handler)
- Minimal working model & routes for food + dispatch  

Here are all the *starter files* for your `client/` and `server/` folders to make CityMap's platform fully functional:

---

✅ CLIENT — React PWA (`client/`)

`client/src/index.js`

```js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./style.css";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);
```

---

`client/src/authHeader.js`

```js
import axios from "axios";

export const getAuthHeader = async () => {
  let token = sessionStorage.getItem("access");
  try {
    await axios.get("/api/admin/ping", {
      headers: { Authorization: `Bearer token` ,
    );
   catch 
    const refresh = localStorage.getItem("refresh");
    const res = await axios.post("/api/admin/refresh",  refreshToken: refresh );
    token = res.data.accessToken;
    sessionStorage.setItem("access", token);
  
  return  headers:  Authorization: `Bearer{token}` } };
};
```

---

`client/src/Login.jsx`

```jsx
import React, { useState } from "react";
import axios from "axios";

export default function Login({ onLogin }) {
  const [username, setUser] = useState("");
  const [password, setPass] = useState("");
  const [error, setErr] = useState("");

  const login = async () => {
    try { 
const res = await axios.post("/api/admin/login", { username, password });
      sessionStorage.setItem("access", res.data.accessToken);
      localStorage.setItem("refresh", res.data.refreshToken);
      onLogin();
    } catch {
      setErr("Login failed");
    }
  };

  return (
    <div>
      <h3>Login</h3>
      <input placeholder="Username" onChange={e => setUser(e.target.value)} />
      <input placeholder="Password" type="password" onChange={e => setPass(e.target.value)} />
      <button onClick={login}>Login</button>
      <p>{error}</p>
    </div>
  );
}
```

---

`client/src/AdminDashboard.jsx`

```jsx
import React, { useState } from "react";
import FoodOrdersAdmin from "./FoodOrdersAdmin";
import DispatchOrdersAdmin from "./DispatchOrdersAdmin";

export default function AdminDashboard() {
  const [tab, setTab] = useState("food");

  return (
    <div>
      <button onClick={() => setTab("food")}>Food Orders</button>
      <button onClick={() => setTab("dispatch")}>Dispatch Orders</button>
      <button onClick={() => {
        sessionStorage.clear();
        localStorage.removeItem("refresh");
        window.location.reload();
      }}>Logout</button>
      {tab === "food" && <FoodOrdersAdmin />}
      {tab === "dispatch" && <DispatchOrdersAdmin />}
    </div> 
);
}
```

---

`client/src/FoodOrdersAdmin.jsx`

```jsx
import React, { useEffect, useState } from "react";
import axios from "axios";
import { getAuthHeader } from "./authHeader";

export default function FoodOrdersAdmin() {
  const [orders, setOrders] = useState([]);

  useEffect(() => {
    const fetchOrders = async () => {
      const config = await getAuthHeader();
      const res = await axios.get("/api/admin/food-orders", config);
      setOrders(res.data);
    };
    fetchOrders();
  }, []);

  return (
    <div>
      <h4>Food Orders</h4>
      {orders.map(order => (
        <div key={order._id}>{order.details}</div>
      ))}
    </div>
  );
}
```

---

`client/src/DispatchOrdersAdmin.jsx`

Same as above but for dispatch orders — just switch the endpoint.

---

✅ SERVER (`server/`)

`server/index.js`

```js
import express from "express";
import mongoose from "mongoose";
import dotenv from "dotenv";
import cors from "cors";
import authRoutes from "./routes/authRoutes.js";
import foodRoutes from "./routes/foodRoutes.js";
import dispatchRoutes from "./routes/dispatchRoutes.js";

dotenv.config();
const app = express();
app.use(cors());
app.use(express.json());

app.use("/api/admin", authRoutes);
app.use("/api/admin", foodRoutes);
app.use("/api/admin", dispatchRoutes);
app.get("/api/admin/ping", (req, res) => res.send("OK"));

mongoose.connect(process.env.MONGO_URI).then(() => {
  app.listen(4000, () => console.log("API running on port 4000"));
});
```

---

`server/models/Admin.js`

```js
import mongoose from "mongoose";

const AdminSchema = new mongoose.Schema({
  username: String,
  passwordHash: String,
  role: { type: String, enum: ["admin", "dispatch", "support"], default: "admin" },
});

export default mongoose.model("Admin", AdminSchema);
```

---

`server/models/FoodOrder.js` and `DispatchOrder.js`

Same structure:

```js
import mongoose from "mongoose";

const FoodOrderSchema = new mongoose.Schema({
  details: String,
  status: String,
}, { timestamps: true });

export default mongoose.model("FoodOrder", FoodOrderSchema);
```

---

`server/middleware/auth.js`

```js
import jwt from "jsonwebtoken";

export const verify = (roles) => (req, res, next) => {
  const token = req.headers.authorization?.split(" ")[1];
  if (!token) return res.status(403).send("No token");

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    if (!roles.includes(decoded.role)) return res.status(403).send("Forbidden");
    req.admin = decoded;
    next();
  } catch {
    res.status(403).send("Invalid token");
  }
};
```

---
`server/routes/authRoutes.js`

```js
import express from "express";
import Admin from "../models/Admin.js";
import jwt from "jsonwebtoken";
import bcrypt from "bcryptjs";

const router = express.Router();

router.post("/login", async (req, res) => {
  const { username, password } = req.body;
  const admin = await Admin.findOne({ username });
  if (!admin) return res.status(401).send("Invalid");
  const match = await bcrypt.compare(password, admin.passwordHash);
  if (!match) return res.status(401).send("Invalid");

  const accessToken = jwt.sign({ username, role: admin.role }, process.env.JWT_SECRET, { expiresIn: "15m" });
  const refreshToken = jwt.sign({ username, role: admin.role }, process.env.JWT_REFRESH_SECRET, { expiresIn: "7d" });

  res.json({ accessToken, refreshToken });
});

router.post("/refresh", (req, res) => {
  const { refreshToken } = req.body;
  if (!refreshToken) return res.status(401).send("No token");
  try {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    const accessToken = jwt.sign({ username: decoded.username, role: decoded.role }, process.env.JWT_SECRET, { expiresIn: "15m" });
    res.json({ accessToken });
  } catch {
    res.status(403).send("Invalid token");
  }
});

export default router;
```

---
`server/routes/foodRoutes.js` and `dispatchRoutes.js`

Use `verify(["admin", "dispatch"])`:

```js
import express from "express";
import FoodOrder from "../models/FoodOrder.js";
import { verify } from "../middleware/auth.js";

const router = express.Router();

router.get("/food-orders", verify(["admin"]), async (req, res) => {
  const orders = await FoodOrder.find().sort({ createdAt: -1 });
  res.json(orders);
});

export




