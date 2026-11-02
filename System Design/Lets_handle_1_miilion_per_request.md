Lets Handle 1 million requests per second
    - aws IAM is busiest route in the world
    - IAM handles 400M requests per second.
    - SQl, Multithreading, Unix, Clustering, Redis, Node.js, c++
    - c++ is best and node.js,java, python not good enough for high stake products.
    - Core utilisation
        - core #1 utilisation + core #2 utilisation + ..... 
        - total time - idle time/total time * 100
    - CPU utilisation
        - (core #1 utilisation + core #2 utilisation + ...../ total no of cores)
        - 
    - Resource Monitoring
    - Spend time thinking about how to handle Race conditions and semaphors
    - Autocannon is npm library to produce heavy load and send it to server

