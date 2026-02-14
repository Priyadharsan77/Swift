# 📍 Geo App — The Complete Guide

## 👋 Part 1: The "Plain English" Guide (For Everyone)

### What is this app?
Imagine you're an explorer. **Geo App** is your digital compass and logbook. It does two main things:
1.  **It knows where you are:** It tracks your movement in real-time.
2.  **It remembers where you've been:** You can drop "pins" on a map to save interesting spots.

Think of it like a simplified version of Google Maps, but built by us!

### How it Works (The Story)
Let's see the app in action:

![App Screenshot 1](images/screenshot_1.png)
![App Screenshot 2](images/screenshot_2.png)

1.  **The Scout (Tracking You):**  
    When you open the app, it immediately starts acting like a scout. It looks at the GPS satellites to figure out your exact location. It doesn't just keep this secret; it constantly radios back to "Base Camp" (our Server) to say, "Hey, I'm at these coordinates right now!"
    
2.  **The Compass (Guiding You):**  
    On the screen, you see a compass. It doesn't just point North; it points to your last known destination. If you turn your body, the arrow turns with you, always guiding you to your target.

3.  **The Map (Your Logbook):**  
    Tap the compass, and a map opens up. This is your canvas. See a cool coffee shop? Tap the map! 
    - **Any tap does two things:**
        - It drops a red pin on that spot.
        - It sends a message to Base Camp: "Save this spot forever!"
    - Now, even if you close the app and come back next week, that pin will still be there.

### The Big Picture
Software is like a restaurant. 

-   **The Frontend (iOS App) is the Dining Room:** This is what you see and touch. It's the menus (buttons), the tables (screens), and the waiters (app logic).
-   **The Backend (Server) is the Kitchen:** This is hidden in the back. It stores the ingredients (Data) and cooks the meals (Processes requests).
-   **The API is the Order Ticket:** It's the language the waiter uses to tell the kitchen what you want.

---

## 🛠️ Part 2: The "Under the Hood" Analysis (For Developers)

This section breaks down exactly **what** each file does and **where** it fits in the machine.

### 📱 Frontend: The iOS App (`geo-app/compass/`)
*Built with Swift & UIKit.*

#### 1. The Entrance
-   **`AppDelegate.swift`**:  
    * **What:** The main entry point. It's the "manager" of the app.  
    * **Where:** It handles the app launching and closing. When you tap the app icon, this code runs first.

#### 2. The Visuals (View Controllers)
-   **`CompassViewController.swift`**:  
    * **What:** The main screen with the big arrow.  
    * **Key Logic:**  
        - It asks `LocationDelegate` "Where am I?".
        - It calculates the angle of the arrow so it points to your destination.
        - **Wired to:** Tapping the screen triggers `showMap()`, which opens the `MapViewController`.
-   **`MapViewController.swift`**:  
    * **What:** The screen showing the map.  
    * **Key Logic:**  
        - **Display:** Shows your blue dot (User Location).
        - **Interaction:** Detects when you tap the screen (`didTap`).
        - **Network Call:** When you tap, it behaves like a waiter taking an order. It creates a `POST` request and sends your tap coordinates to the Backend (`http://localhost:3000/api/v1/pins`).

#### 3. The Logic & Data
-   **`LocationDelegate.swift`**:  
    * **What:** The GPS Handler.  
    * **Key Logic:** It acts like a background sensor. Every time you move, this file runs.  
    * **Wired to:** It quietly sends a `PUT` request to the Backend (`/api/v1/user`) to update your live location on the server.
-   **`Models/` Folder**:  
    * **`Pin.swift`, `Pinresponse.swift`, `Location.swift`**:  
    * **What:** These are "Blueprints". They tell the app what data looks like. For example, "A Pin must have an ID and coordinates."

### 🧠 Backend: The Nervous System (`geo-app-backend/`)
*Built with Node.js, Express & MongoDB.*

#### 1. The Command Center
-   **`server.js`**:  
    * **What:** The brain of the operation.  
    * **Key Logic:** It starts the server, connects to the database, and sets up the "Doorways" (Routes) for data to come in.
-   **`config/db.js`**:  
    * **What:** The specific code that knows the password and address to connect to our MongoDB database.

#### 2. The Receptionists (Routes)
*Think of these as signposts directing traffic.*
-   **`routes/pins.js`**:  
    * **What:** Handles traffic for `/api/v1/pins`.  
    * **Wired to:** If a request comes here, it forwards it to the `pinsController`.
-   **`routes/user.js`**:  
    * **What:** Handles traffic for `/api/v1/user`.  
    * **Wired to:** Forwards requests to the `userController`.

#### 3. The Workers (Controllers)
*This is where the actual work happens.*
-   **`controllers/pins.js`**:  
    * **What:**  
        - `getPins`: Goes to the database, grabs all pins, and sends them back.
        - `addPins`: Takes the coordinates sent from the iPhone and saves them to the database.
-   **`controllers/user.js`**:  
    * **What:**  
        - `updateUser`: Takes your latest GPS location and overwrites your old location in the database.

#### 4. The Filing Cabinets (Models)
*These define how data is stored in MongoDB.*
-   **`models/Pins.js`**:  
    * **What:** Defines that a "Pin" looks like `{ pinId: String, location: [Number] }`.
-   **`models/User.js`**:  
    * **What:** Defines that a "User" looks like `{ userId: String, location: [Number] }`.

---

## ⚡ Data Wiring Diagram
*How it all connects.*

### Scenario A: You Drop a Pin
1.  **👆 You Tap** (`MapViewController.swift`)
2.  **📡 App Sends** (`POST /api/v1/pins`)
    ↓
3.  **🖥️ Server Receives** (`server.js`)
4.  **🔀 Route Directs** (`routes/pins.js`)
5.  **⚙️ Controller Actions** (`controllers/pins.js` -> `addPins`)
    ↓
6.  **💾 Database Saves** (`MongoDB`) -> **✅ SUCCESS!**

### Scenario B: You Walk Down the Street
1.  **🚶 You Move** (`GPS Satellite Update`)
2.  **📱 App Detects** (`LocationDelegate.swift`)
3.  **📡 App Sends** (`PUT /api/v1/user`)
    ↓
4.  **🖥️ Server Receives** (`server.js`)
5.  **🔀 Route Directs** (`routes/user.js`)
6.  **⚙️ Controller Updates** (`controllers/user.js` -> `updateUser`)
    ↓
7.  **💾 Database Updates** (`MongoDB`) -> **✅ LIVE TRACKING!**

---

## 🚀 Quick Start Guide (Simplified)

### 1. Start the Brain (Backend)
Open your terminal (Command Line) and run these 3 magic spells:
```bash
brew services start mongodb-community   # Start the Database
cd geo-app-backend
npm run dev                             # Start the Server
```
*You should see: "Server running in development mode on port 3000"*

### 2. wake up the Body (Frontend)
1.  Open `Compass.xcodeproj` in Xcode.
2.  Select a **Simulator** (like iPhone 16) from the top bar.
3.  Press the **Play ▶️** button.

### 3. Test It
- In the Simulator, go to **Features -> Location -> City Run** to simulate movement.
- Watch your backend terminal — you'll see your location updating in real-time!
