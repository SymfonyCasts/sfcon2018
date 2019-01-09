# Leverage the power of Symfony components within ApiPlatform

by Antoine Bluchet

ApiPlatform is great for building API-first software. Even better, it's build on top of Symfony! Did you hear about Symfony's Workflow component? Or the brand new Messenger component? They offer powerful tools!

For example, a pizza order can take multiple statuses, each one in transition with another (order, pay, wait, eat). This is a workflow.

While waiting for the product, it needs to be cooked! Every time the order is paid, we have to send an instruction to the kitchen. Today, why don't we automatically push that message?

Through a real-life use case, I will demonstrate how well these two fit on top of our Symfony-based API!