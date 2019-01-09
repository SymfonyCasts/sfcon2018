# Using Symfony Forms with Rich Domain Models

by Christopher Hertel & Christian Flothmann

With the popularization of DDD people started shifting from anemic models with only getters and setters to a rich model describing the state changes in specific methods. This way of designing models does not play well with Symfony forms. User provided input is inherently invalid while we want to maintain certain invariants in our domain model. A common approach to overcome these limitations is to create data transfer objects our forms are then bound to. This can lead to lots of mapping & glue code that might be cumbersome to write and maintain.

But couldn’t we do better? In this talk we will discuss the different aspects of a rich domain model that makes it hard to use it in conjunction with the Form component. We will then look at the possibilities to hook into the data flow of the form handling and discover how we can modify it to interact seamlessly with our model.