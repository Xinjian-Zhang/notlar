## Software Quality and Standards Lab 11 – Microservices

### Exercise 1:Reproducible builds (unfinished from Lab 10).

Areproducible buildis the process of compiling software in a way that repeated builds yield the same output. This technique aims to make output of the build process more predictable by (1) identifying exact versions of inputs that go into the build and (2) observing how the state of the build environment affects it. With growing dependencies on external inputs, managing inputs in this way is becoming increasingly important. In adeterministic build, the build environment is immutable, and the same inputs always yield the same output.

(a) What are sources of non-determinism in software builds? Give examples of “moving parts” of the environment that affect the output of a software build.

(b) Find examples of techniques to improve reproducibility of software builds.Hint: Lock-files.

(c) How does increasing reproducibility of builds improve software quality? Which risks does it mini-mize? Explain how.Hints: Supply-chain Attacks, Source Availability.

(d) There are standards and tools to help make software more reproducible. Go read about them.

- Read what the Linux kernel does to achieve reproducible builds in the Kernel Documentation.
- The operating system NixOS powered by a (purely functional) package manager that enforces reproducible builds. You can read about How Nix works. The same programming language that is used to build software can be used for more complex tasks, such as orchestrating declarative  deployments to the cloud (with NixOps). Since builds are fully deterministic, outputs can be  cached and shared, speeding up CI/CD by preventing unnecessary rebuilds
- The configuration language Dhall attempts to prevent copy-and-paste errors in configuration  files by establishing a single source of truth. You can use it to write manifests for Kubernetes.  Configurations become reproducible when they declare a “semantic hash value” of their inputs:  the hash value changes only when the input changes behavior. In particular, one can be sure  that refactoring an input did not change behavior by checking that the semantic hash value of  old and new code are the same. Read more about it here.

----

### Exercise 2:Microservices.


(a) Recall from the lecture what areadvantagesandchallengesof a microservice-based architecture when compared to software developed as a monolith.

(b) How does a product implemented via microservices affect theresponsibilitiesof different roles (man-ages, developers, architects,... ) in a project? Compare with other architectures.

(c) The (fictional) online thrift store “Duckbucks” of entrepreneur Scrooge McDuck is seeking to migrate from their monolith to a microservices-based architecture. The online store allows customers to place request to buy and sell specific items, and those are searchable on a web app. The back-end is built around one central database storing all buy/sell requests, as well as all user accounts. Scrooge wants match buyers and sellers if it is likely that one is looking for an item that the other is selling: For example, a buyer looking for “1920s Czech glassware” might be a good match for a seller offering “antique Bohemian crystal”. Right now, the infrastucture is too rigid to implement this efficiently. He hopes that by gradually decoupling the system, he can integrate an ML-backed system that can perform the matching. Propose a microservice architecture for the online store. Describe what the individual services are, and how they interface with each other. Focus on core functionality first, then think about how to
integrate the new ML-backed matching system. Make sure to only describe the overall architecture, not the implementation of individual services — that’s a job for the developers. For an added challenge, provide a plan how to migrate away from the monolith step-by-step.


