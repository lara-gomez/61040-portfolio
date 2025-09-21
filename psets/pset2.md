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

1. Design a couple of additional concepts to realize this extension, and write them out in full (but including only the essential actions and state). It should not be necessary to make any changes to the existing concepts.

**concept** User[UrlShortening]

**purpose** support multiple users and manage the shortened urls they have registered

**principle** after registering a url, associate the url with the user who created this shortening

**state** 

    a set of Users with

        a set of registeredUrls UrlShortenings

**actions** 

    associateShortening (user: User, shortUrl: String): (registeredUrl: UrlShortening)

        requires: the user exists and the shortUrl has been created

        effects: adds the shortUrl to the set of registeredUrls for the given user, returns the shortened url

    checkOwner (user: User, shortUrl: String): (check: Flag)

        requires: user exists

        effects: returns true if the shortUrl is registered by the given user; false otherwise


**concept** Analytics[UrlShortening]

**purpose** access information about how short links are used

**principle** keep track of the count of accesses for a short url, such that every time a user accesses the url, the access count increases. Only the user who registered the shortening should be able to view the total count.

**state**

    a set of shortUrls UrlShortenings with

        an accessCount Number

**actions**

    accessAnalytics (user: User, shortUrl: String): (accessCount: Number)

        requires: the user and shortUrl exist and the given user registered the shortUrl

        effects: return the accessCount for the shortUrl

    increaseAccessCount (shortUrl: String): (accessCount: Number)

        requires: shortUrl exists

        effects: accessCount += 1, return the updated count

2. Specify three essential synchronizations with your new concepts: one that happens when shortenings are created; one when shortenings are translated to targets; and one when a user examines analytics.

**sync** create

**when** 

    Request.shortenUrl (user: User)
    UrlShortening.register (): (shortUrl)

**then** User.associateShortening (user, shortUrl)


**sync** translate

**when** UrlShortening.lookup (shortUrl)

**then** Analytics.increaseAccessCount (shortUrl)


**sync** examineAnalytics

**when** Request.accessAnalytics(user, shortUrl)

**then** Analytics.accessAnalytics (user, shortUrl)

3. As a way to assess the modularity of your solution, consider each of the following feature requests, to be included along with analytics. For each one, outline how it might be realized (eg, by changing or adding a concept or a sync), or argue that the feature would be undesirable and should not be included:

- Allowing users to choose their own short URLs

    Within UrlShortening, the register action already takes in all the parameters to support a custom short URL. The existing register sync calls NonceGeneration.generate () to produce the nonce that will be used as the shortUrlSuffix for UrlShortening.register. We will add another sync called createCustomLink, such that when Request.shortenUrl (targetUrl, shortUrlBase, shortUrlSuffix), then UrlShortening.register (shortUrlSuffix, shortUrlBase, targetUrl). This way, there is support for both randomized nonces for the short url suffix or support for completely customized short URLs that users may prefer. This modification is already supported by the requires section for UrlShortening.register since it ensures uniqueness.

- Using the “word as nonce” strategy to generate more memorable short URLs

    To implement the word as nonce strategy, we will modify the NonceGeneration concept. The state would include a set of Contexts, each with its own used set of Strings. In addition, we can maintain a global set of Words, which are strings representing memorable words we may use as nonces. NonceGeneration.generate() would then return a word from Words that is not in the context's used set to preserve uniqueness, then add that word to the used set. Since we are only modifying one function of NonceGeneration internally, the change remains modular given that the type of the output is still represented the same way.

- Including the target URL in analytics, so that lookups of different short URLs can be grouped together when they refer to the same target URL

    We will extend our Analytics concept to include the target URL. Since both the short URL and the target URL are strings, we will generalize the state to maintain a single set of Url Strings, each with an accessCount number. Both actions for Analytics will take in any URL type and increase the access count or return the count. In the translate sync defined above, use the targetUrl return from UrlShortening.lookup (shortUrl) to increase the access count for both the shortUrl and the targetUrl.

- Generate short URLs that are not easily guessed

    Since we decided to use the word as nonce strategy above, we are choosing to prioritize usability and clarity over the security benefits provided by hard-to-guess short URLs. This design is less intuitive for users, and more prone to typos that may decrease the quality of user experience. It may potentially defeat the purpose of short URLs to begin with, making it more difficult to access the not easily guessable link over the original target URL.

- Supporting reporting of analytics to creators of short URLs who have not registered as user

    It is difficult to manage analytics for creators who have not already registered as a user. There is no association that can readily provide whether they are confirmed as a creator of the short URL, making it unreasonable to provide analytics for such creators. In the same vein, a user would most likely want to manage all of their short URLs in one place, which is better supported through a registered account. In the design for this feature, we would have to duplicate the functionality of the User, thus violating modularity by splitting the same logic across two concepts.

