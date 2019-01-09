# Going crazy with Varnish: Caching pages of logged in users

by David Buchmann

You know how HTTP caching works but need more? In this talk we look into ways to cache personalized content. We will look at Edge Side Includes (ESI) to tailor caching rules of fragments, and at the user context concept to differentiate caches not by individual user but by permission groups. A big help to leverage this concept is the FOSHttpCache in combination with either Varnish or the Symfony HttpCache reverse proxy.