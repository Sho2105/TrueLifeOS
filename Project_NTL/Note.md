# Indie Game Code Architecture: Best Practices

## 1. The "Lego Brick" Philosophy (Modular Systems)
* **Isolated Modules:** Build systems (health, movement, spawners) to be completely independent from one another [00:01:03].
* **The 70/30 Rule:** Aim for roughly 70% reusable "bricks" and 30% "glue code" [00:03:32].
* **Rule of Thumb:** Write every system with the intent to reuse it in a future project, even if you don't. This keeps the code tidy and modular [00:05:24].

## 2. Connecting Systems with "Glue"
* **Avoid Direct References:** Do not let a movement system reference a health system directly. This creates a "monolith" that can't be separated [00:01:43].
* **Dedicated Glue Code:** Use specialized scripts, game managers, or system connectors that exist solely to tie components together [00:02:03].
* **Small Game Managers:** Instead of one massive "Master Manager," break them into tiny, specialized tasks (e.g., Day/Night manager, Building manager) [00:06:01].

## 3. Separation of Data and Code
* **Immutable Data:** Keep balancing parameters, translations, and game design numbers in a separate "database" (spreadsheets, scriptable objects, or JSON) [00:07:29].
* **Usability:** Storing all design data in one place makes it easier to update, expand, and—critically—makes the game much more accessible for **modding** [00:10:12].
* **Database-First Design:** Designing the data structure before writing the systems ensures a clean, predictable architecture [00:10:34].

## 4. Controlling "Time Spaghetti" (Execution Order)
* **The Execution Order Problem:** Default engine execution can be random and "ugly" for complex projects [00:12:24].
* **Manual Update Loops:** For gameplay-relevant logic, use a master script that calls update events in a strict, controlled order to ensure stability [00:11:47].

## 5. Simulation vs. View
* **The Logic Layer (Simulation):** This is the "raw data" and bounding boxes. It shouldn't care about (or even know) that visuals exist [00:13:43].
* **The Visual Layer (View):** This layer simply observes the simulation and renders it. It treats the simulation as a real-time database [00:13:25].
* **Independence:** Separating art from gameplay logic allows you to replace visuals without breaking the core game state [00:14:19].

## Final Reminder
* **If it works, it works:** Most games are "pure spaghetti" under the hood. Perfection isn't the goal; organization that fits the specific needs of your project is [00:15:09].
