# Problem Set 2: Composing Concepts

## Concept Questions

1. Contexts. The NonceGeneration concept ensures that the short strings it generates will be unique and not result in conflicts. What are the contexts for, and what will a context end up being in the URL shortening app?

    In the NonceGeneration concept, we can guarantee unique nonces through the use of the context. This is due to the used set of strings being tied to one given context. In the URL shortening app, the context represents the short URL base, or the domain. We would want to maintain a list of unique suffixes within a given domain (context), while still allowing the suffix to be available for use under a different domain.


2. Storing used strings. Why must the NonceGeneration store sets of used strings? One simple way to implement the NonceGeneration is to maintain a counter for each context and increment it every time the generate action is called. In this case, how is the set of used strings in the specification related to the counter in the implementation? (In abstract data type lingo, this is asking you to describe an abstraction function.)

    Abstraction function: given a context, generate a unique key such that all generated keys within a context are tracked within a used set of strings. There are multiple ways to accomplish this, such as maintaining a used set of used strings as in our current implementation. The counter is another way to accomplish the goal of our abstraction function through the implied way of ensured uniqueness.

3. Words as nonces. One option for nonce generation is to use common dictionary words (in the style of yellkey.com, for example) resulting in more easily remembered shortenings. What is one advantage and one disadvantage of this scheme, both from the perspective of the user? How would you modify the NonceGeneration concept to realize this idea?

    From the perspective of the user, using common dictionary words is advantageous since they are readable and minimize the amount of typos that could occur when looking up the url. A disadvantage is that there are limited words in the dictionary, so producing a unique word may lead to longer or more complicated words being generated, which are equally as unintuitive as a random string. If we wanted to realize this idea within the NonceGeneration concept, we would add a set of dictionary words to the set of contexts in the state. In the generate action, we would return a word from the dictionary that is not already used by the context, and add it to the used set. In the case that we have already used all the words in the dictionary, then combine words.


## Synchronization Questions

1. Partial matching. In the first sync (called generate), the Request.shortenUrl action in the when clause includes the shortUrlBase argument but not the targetUrl argument. In the second sync (called register) both appear. Why is this?

    The generate sync is only related to nonce generation, which takes in a single parameter (context). As discussed above, the context is associated with the shortened URL base. Therefore, the only relevant parameter that we need from the request is the shortUrlBase. In contrast, the register sync invokes the UrlShortening.register action, which requires a shortUrlSuffix, shortUrlBase, and targetUrl. Since the generate sync will call the NonceGeneration.generate action, we will grab the shortUrlSuffix here. This means the request must provide us with the shortUrlBase and targetUrl in order to create a valid call to the register action. By only including goal-specific arguments for each sync, we provide design oriented, concise syncs which avoid too much detail about implementation.

2. Omitting names. The convention that allows names to be omitted when argument or result names are the same as their variable names is convenient and allows for a more succinct specification. Why isn’t this convention used in every case?

    Although argument names can be omitted when they are the same as their variable names, there are situations where not using this convention is more useful for readers. For example, the use of partial matching may result in a disconnect between which variables we are actually using. In addition, the name of the state for a concept is not always explicitly related to the input, such as in the case of context within NonceGeneration. Mapping shortUrlBase to context in the generate sync thus clearly states the connection. Displaying the argument names and their types sometimes explicitly improves readability and clarity, which are more important from a user perspective.

3. Inclusion of request. Why is the request action included in the first two syncs but not the third one?

    The ExpiringResource.setExpiry action is triggered by the UrlShortening.register action, and it only takes in a resource and seconds arguments. It acquires the resource argument from the return of the register action, so it does not need any direct user input to produce the setExpiry effect. In contrast, the first two syncs are consequences of user interaction to actually link a shortUrl to a targetUrl so they require the user to manually input their desired target and shortening information. Therefore, they must include the request action.

4. Fixed domain. Suppose the application did not support alternative domain names, and always used a fixed one such as “bit.ly.” How would you change the synchronizations to implement this?

    In this case, the shortUrlBase parameter is no longer needed. The NonceGeneration.generate action would no longer need a set of Contexts. We would have a single used set of strings instead, so we would not have to take in the shortUrlBase from the request or as an argument. For the register sync, the request would only need the targetUrl for Request.shortenUrl, and the "then" section would update to UrlShortening.register (shortUrlSuffix: nonce, targetUrl). The setExpiry sync would remain unchanged.

5. Adding a sync. These synchronizations are not complete; in particular, they don’t do anything when a resource expires. Write a sync for this case, using appropriate actions from the ExpiringResource and URLShortening concepts.

    **sync** deleteExpiry

    **when** ExpiringResource.expireResource(): (resource: shortUrl)

    **then** UrlShortening.delete(shortUrl: resource)


## Extending the Design

