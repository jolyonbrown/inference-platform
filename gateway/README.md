# Gateway

The gateway is a deliberately small control layer in front of `vLLM`.

It exists to make the project feel like a platform rather than a raw model deployment.

Planned responsibilities:

- request validation
- token budget enforcement
- request classification
- queue depth metrics
- controlled rejection under overload
