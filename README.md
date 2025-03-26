# BambangShop Publisher App
Tutorial and Example for Advanced Programming 2024 - Faculty of Computer Science, Universitas Indonesia

---

## About this Project
In this repository, we have provided you a REST (REpresentational State Transfer) API project using Rocket web framework.

This project consists of four modules:
1.  `controller`: this module contains handler functions used to receive request and send responses.
    In Model-View-Controller (MVC) pattern, this is the Controller part.
2.  `model`: this module contains structs that serve as data containers.
    In MVC pattern, this is the Model part.
3.  `service`: this module contains structs with business logic methods.
    In MVC pattern, this is also the Model part.
4.  `repository`: this module contains structs that serve as databases and methods to access the databases.
    You can use methods of the struct to get list of objects, or operating an object (create, read, update, delete).

This repository provides a basic functionality that makes BambangShop work: ability to create, read, and delete `Product`s.
This repository already contains a functioning `Product` model, repository, service, and controllers that you can try right away.

As this is an Observer Design Pattern tutorial repository, you need to implement another feature: `Notification`.
This feature will notify creation, promotion, and deletion of a product, to external subscribers that are interested of a certain product type.
The subscribers are another Rocket instances, so the notification will be sent using HTTP POST request to each subscriber's `receive notification` address.

## API Documentations

You can download the Postman Collection JSON here: https://ristek.link/AdvProgWeek7Postman

After you download the Postman Collection, you can try the endpoints inside "BambangShop Publisher" folder.
This Postman collection also contains endpoints that you need to implement later on (the `Notification` feature).

Postman is an installable client that you can use to test web endpoints using HTTP request.
You can also make automated functional testing scripts for REST API projects using this client.
You can install Postman via this website: https://www.postman.com/downloads/

## How to Run in Development Environment
1.  Set up environment variables first by creating `.env` file.
    Here is the example of `.env` file:
    ```bash
    APP_INSTANCE_ROOT_URL="http://localhost:8000"
    ```
    Here are the details of each environment variable:
    | variable              | type   | description                                                |
    |-----------------------|--------|------------------------------------------------------------|
    | APP_INSTANCE_ROOT_URL | string | URL address where this publisher instance can be accessed. |
2.  Use `cargo run` to run this app.
    (You might want to use `cargo check` if you only need to verify your work without running the app.)

## Mandatory Checklists (Publisher)
-   [x] Clone https://gitlab.com/ichlaffterlalu/bambangshop to a new repository.
-   **STAGE 1: Implement models and repositories**
    -   [x] Commit: `Create Subscriber model struct.`
    -   [x] Commit: `Create Notification model struct.`
    -   [x] Commit: `Create Subscriber database and Subscriber repository struct skeleton.`
    -   [x] Commit: `Implement add function in Subscriber repository.`
    -   [x] Commit: `Implement list_all function in Subscriber repository.`
    -   [x] Commit: `Implement delete function in Subscriber repository.`
    -   [x] Write answers of your learning module's "Reflection Publisher-1" questions in this README.
-   **STAGE 2: Implement services and controllers**
    -   [x] Commit: `Create Notification service struct skeleton.`
    -   [x] Commit: `Implement subscribe function in Notification service.`
    -   [x] Commit: `Implement subscribe function in Notification controller.`
    -   [x] Commit: `Implement unsubscribe function in Notification service.`
    -   [x] Commit: `Implement unsubscribe function in Notification controller.`
    -   [x] Write answers of your learning module's "Reflection Publisher-2" questions in this README.
-   **STAGE 3: Implement notification mechanism**
    -   [ ] Commit: `Implement update method in Subscriber model to send notification HTTP requests.`
    -   [ ] Commit: `Implement notify function in Notification service to notify each Subscriber.`
    -   [ ] Commit: `Implement publish function in Program service and Program controller.`
    -   [ ] Commit: `Edit Product service methods to call notify after create/delete.`
    -   [ ] Write answers of your learning module's "Reflection Publisher-3" questions in this README.

## Your Reflections
This is the place for you to write reflections:

### Mandatory (Publisher) Reflections

#### Reflection Publisher-1

1. **Do we need an interface (or trait in Rust) in BambangShop, or is a single Model struct enough?**

    In the Observer pattern, an interface (or trait in Rust) defines a contract that all subscribers must follow, ensuring that different implementations can coexist. In Rust, traits allow polymorphic behavior and ensure that all observers implement necessary methods, such as `update` on the example that I have recently added for Publisher-1.

    Now, in the **BambangShop case**, we currently use a `Subscriber` struct without a trait. This is acceptable if we assume that all subscribers have the same structure and behavior. However, if we later introduce different types of subscribers (e.g., email subscribers, webhook subscribers), a trait would be beneficial to define a shared behavior, such as:

    ```Rust
    trait Subscriber {
        fn notify(&self, notification: &Notification);
    }
    ```

    Currently, since we have only one `Subscriber` struct, a trait is not strictly necessary, but it would improve flexibility, maybe throughout the tutorial, we will se it being used.

2. **Is using Vec (list) sufficient, or is DashMap necessary?**

    In my own opinion, since the `id` in **Program** and `url` in **Subscriber** must be unique, meaning a key-value like storage (hashmap/dictionary) structure is ideal.

    a. Using `Vec<Subscriber>` has its own cons like:
    -   Searching for a subscriber by `url` requires O(n) time complexity (sorry I did this after DAA)
    -   Removing a subscriber also requires iteration
    -   If the number of subscribers too much, it can be inefficient

    b. Instead, if we use `DashMap<String, Subscriber>` (Map/Dictionary) can be beneficial because:
    -   Allows constant time (O(1)) for lookups, insertions, and deletions by using the `url` as the key.
    -   By the property of dictionary, a key must be unique, so this makes our `url` unique

    Since we need efficient lookups, `DashMap` is the better choice for managing subscribers.

3. **Do we need DashMap, or can we implement Singleton instead?**

    The Singleton pattern ensures a single global instance, which we already achieve with `lazy_static!`:

    ```Rust
    lazy_static! {
    pub static ref SUBSCRIBERS: DashMap<String, Subscriber> = DashMap::new();
    }
    ```

    However, Singleton alone does not ensure thread safety. `DashMap` in Rust is specifically designed for concurrent access, avoiding data races without requiring manual locking.

    -   IF we remove `DashMap` and use Singleton with `HashMap`, we would ened to wrap it in a `Mutex` or `RwLock`, as a locking overhead

    -   Since `DashMap` provides both singleton behavior and thread safety, it is the best choice

    In conclusion, `DashMap` should be kept instead of replacing it with just a Singleton.

#### Reflection Publisher-2

1. **Why do we need to separate “Service” and “Repository” from the Model?**

    In MVC, the Model handles both business logic and data access, but there are some best practices advocate that people usually do for separating these concerns

    a. Separation of Concerns (SoC):
    -   Repository: Handles database access, queries, and persistence
    -   Service: Contains business logic and orchestrates operations
    -   Model: Represents data structure but doesn't handle business logic or database interactions
    b. Scalability & Maintainability:
    -   If we mix business logic and data access in the Model, it becomes difficult to refactor, test or extend.
    -   A dedicated Service layer allows adding additional business rules without modifying the database logic.
    -   A Repository layer abstracts database implementation, making it easier to switch databases (e.g., from PostgreSQL to MongoDB).
    c. Testability:
    -   The service layer can be unit tested without needing an actual database.
    -   The repository can be mocked for testing, aviding database dependencies in tests.

    In **BambangShop**, the introduction of `NotificationService` and `SubscriberRepositor` follows modern layered architecture, ensuring the system is modular and maintanable.

2. **What happens if we only use the Model?**

    If we don't separate the **Service** and **Repository** layers, the **Model** would handle both business logic and database operations. They could get intertwined, leading to tightly coupled code that is harder to maintain, test, extend, or even reuse. Other things that could happen are but not limited to:
    
    a. Tightly Coupled Components:
    -   The `Notification`, `Subscriber`, and `Program` models would need to directly interact with the database
    -   Any change in business logic would require modifying the Model itself, making the code harder to manage
    b. Complex Interactions Between Models:
    -   `Notification` needs to be sent when a `Subscriber` subscribes/unsubscribes.
    -   Without a Service layer, `Notification` would have to directly interact with the `Subcriber` model, increasing dependencies. This would also make code harder to refactor and debug'
    c. Difficult to Extend Features
    -   If we want to add let's say logging, validation, we would ahve to modify the Model, this could lead to violations of the Single Responsibility Principle (SRP) that we have learned beforehand.

    Thus, without a Service and Repository layer, the codebase becomes more complex, harder to maintain, and less scalable. 

3. **How does Postman help in testing our current work?**

    Postman in a tool for testing APIs. In BambangShop, it could help us on:
    
    **Testing API Endpoints**
    
    - We can send `POST` requests to:
        -   **Subscribe:** `/subscribe/<product_type>`
        -   **Unsubscribe:** `/unsubscribe/<product_type>?<url>`
    - Checks if responses match expectations (e.g, check whether the HTTP status and JSON payloads is correct or not)

    **Useful Features in Postman**
    
    - Collections & Environments:
        -   We can group API requests into collections, just like what we did on this Tutorial
        -   Define variables to switch between local development and production environments

    - Automated Testing (Tests Tab):
        -   Write scritps to validate API responses automatically
    
    - Mock Servers:
        -   Simulate API responses to test frontend behavior without requiring a backend
    
    - API Documentation:
        -   Postman can generate interactive API documentation, making it easier for team members to understand endpoints.

#### Reflection Publisher-3
