The **Left Brain UI — Spatial Neural Node** is a novel **spatial command shell** designed to act as a fluid "skin" for accessing all system tools and intelligence [1, 2]. It is a high-compute VLA interface that treats the **Crystal Touchball (chat)** as the central "gravity source" around which all other capabilities orbit [3].

### **Core Interface Mechanics**
*   **The "/" Command:** Acts as a universal context switch, combining the functionality of a system search (Spotlight) with slash commands. For example, typing `/camera` surfaces the camera node, while `/code` opens the editor skin [3].
*   **Floating Neural Nodes:** These represent recently used apps or tools as **proximity-weighted orbs**. Their placement mimics **spatial memory**:
    *   **High Frequency:** Most-used nodes orbit closest to the center.
    *   **Low Frequency:** Rarely-used nodes drift to the periphery, become fainted, or fade to low opacity [1, 3].
*   **Context Capsules:** Each node is not merely an app window but a "capsule" containing the tool’s recent state and its historical connections to other nodes it interacted with [3].

### **UX & Performance Design**
*   **Async (Optimistic) UI:** To ensure the interface feels responsive even when the **System 2 Guardian** is evaluating an action, the UI uses an optimistic pattern [4]. 
    *   When a user submits a request, the UI **immediately** shows the action as pending with a "glow or pulse animation" [4].
    *   If System 2 approves, the state resolves; if rejected, the UI visually **snaps back** with a structured `reason_code` badge [4].
*   **Spatial Gravity Implementation:** For the MVP, these nodes are rendered as SVG/Canvas elements using CSS physics or libraries like **Framer Motion** to simulate gravitational pull and spring animations [5].

### **Latency Management**
While an RPi might result in a 2–8 second evaluation delay (creating a "blocking" UI feel), the use of a **Jetson AGX Orin** reduces this latency to **under 0.3 seconds**, allowing the spatial interface to operate in near-real-time [1, 6].

Would you like me to begin drafting the **Framer Motion physics configuration** for these nodes so you can implement them in the `/ui-mockup/` directory?