# Course Reflection
**Author:** peter 
**Date:** October 30, 2026

The most challenging concept encountered during the course was understanding cache invalidation strategies and maintaining consistency across a highly distributed architecture. It was easy to conceptually grasp how a Content Delivery Network or an in-memory cache speeds up reads, but managing the operational friction of stale data when a state changes was deeply complex. I overcame this by methodically building minimal code architectures, simulating heavy write loads, and testing how explicit key deletion versus time-to-live expiration policies behave under stress.

Based on initial peer review feedback of my core capstone module, I would drastically improve the error propagation and database transaction rollback mechanics. My early iterations leaned too heavily on assuming successful paths, leaving room for potential race conditions if payment gateways timed out halfway through processing. Refactoring this to implement cleaner atomic rollbacks ensures higher resilience.

Moving forward, I will explore the deep complexities of event-driven messaging architectures, specifically focusing on streaming platforms like Apache Kafka. Learning how to decouple microservices via message brokers will help me build asynchronous background workers capable of managing heavy workflows like batch email notifications and distributed invoice generation cleanly.
