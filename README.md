1. Architectural Philosophy: Microservices vs. Monolith
In a traditional Monolithic hotel system, the UI, business logic, and database access are packaged into a single unit. In our Microservices approach:

Decomposition: We break the system into bounded contexts (User, Booking, Inventory).

Independence: Each service can be written in a different version of Java, use its own database, and be deployed without restarting other services.

Data Sovereignty: Each service "owns" its data. The Booking service cannot directly access the User database; it must request data via an API.

2. Core Design Patterns
A. Service Discovery (The "Dynamic Registry")
In a cloud environment, IP addresses of services are not static.

The Problem: How does the Booking service know the IP of the Inventory service if it changes after a restart?

The Solution: Netflix Eureka. Each service acts as a client that registers its name and location to the Eureka Server. Services look up their "peers" by name (e.g., inventory-service) rather than hardcoded IPs.

B. API Gateway Pattern
The Gateway acts as a reverse proxy.

Routing: It shields the internal microservice URLs from the client (Mobile/Web).

Cross-Cutting Concerns: Instead of implementing security and rate-limiting in every service, we do it once at the Gateway.

Load Balancing: The Gateway works with the Service Registry to distribute incoming traffic evenly across multiple instances of a service.

C. Circuit Breaker Pattern (Resilience)
Distributed systems are prone to partial failures.

Cascading Failure: If the Payment service is slow, the Booking service threads might hang waiting for a response, eventually crashing the entire system.

The Solution: Resilience4j. It monitors calls. If failures exceed a threshold, the "circuit opens," and calls fail fast or return a fallback (e.g., "Payment is processing manually") without stressing the failing service.

3. Communication Strategies
Synchronous (Request/Response)
We use REST via OpenFeign. Feign is a declarative web service client. You define an interface, and Spring Cloud generates the implementation. This makes inter-service calls look like local Java method calls.

Best for: Operations that need an immediate result (e.g., "Is this user authorized?").

Asynchronous (Event-Driven)
We use RabbitMQ to achieve "Eventual Consistency."

The Flow: When a booking is confirmed, the Booking Service publishes a Booking_Created event to an exchange. It doesn't care who is listening.

The Consumer: The Notification service listens for that specific event and sends an email. If the Notification service is down, the message stays in the queue until it comes back online.

4. Data Management: The Database per Service
Each service has its own dedicated database. This prevents Database Coupling.

User Service: Stores credentials and roles.

Inventory Service: Stores room counts, types, and real-time availability.

Booking Service: Stores reservation records, check-in dates, and total costs.

5. Observability and Monitoring
Distributed Tracing
In a monolith, we just check one log file. In microservices, a request might fail in the 3rd service in a chain.

Micrometer/Zipkin: We assign a Trace ID to every incoming request. As the request moves through various services, each log entry carries that ID. We can then "stitch" the request's journey together in a visual UI.

Centralized Logging
Instead of logging into 5 different Docker containers, logs are pushed to a central location (like the ELK stack or a central log file) to simplify debugging.

6. The Deployment Model (Dockerization)
We use Containerization to solve the "it works on my machine" problem.

Dockerfile: Defines the environment (Java 17, Alpine Linux) and the JAR file.

Docker Compose: A script that launches the databases, the message broker, and the microservices in the correct order, creating a private virtual network for them to communicate.
