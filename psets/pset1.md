# Problem Set 1: Reading and Writing Concepts

## Exercise 1: Reading a Concept

### Questions
1. Invariants. What are two invariants of the state? (Hint: one is about aggregation/counts of items, and one relates requests and purchases). Say which one is more important and why; identify the action whose design is most affected by it, and say how it preserves it.

    - The invariants of the state are that the aggregates/counts of the items must always be at least 0, and a purchase for an item can only be made if there is already an existing request for it. The positive item count is most important, since we should not allow support for a negative amount of items in the request as it could represent the owner owing an item to the purchasers or overpurchasing. The purchase action is most affected by this invariant given that it is the only action which decreases the count for any given item. It preserves the invariant through the requirement that the item request must have a count at least equal to the amount it will decrement it by.

2. Fixing an action. Can you identify an action that potentially breaks this important invariant, and say how this might happen? How might this problem be fixed?

    - The remove action potentially breaks this invariant since purchases might have been made before the removal. Although the request is removed, this does not eliminate purchases that were previously made, thus contributing to a negative count for that item. This could be fixed by only allowing the remove action for items that do not have any purchases attached to it already.

3. Inferring behavior. The operational principle describes the typical scenario in which the registry is opened and eventually closed. But a concept specification often allows other scenarios. By reading the specs of the concept actions, say whether a registry can be opened and closed repeatedly. What is a reason to allow this?

    - The specs of the concept actions seem to allow opening and closing the registry repeatedly since each of their effects enable the requirements for the other. Since opening and closing the registry relates to the public visibility, you may want to allow this functionality to avoid accidentally closing the registry. It may be useful to close the registry when the recipient wants to add or remove item requests so that purchasers do not see inaccurate information while it is being modified.

4. Registry deletion. There is no action to delete a registry. Would this matter in practice?

    - Since we never delete registries, this means we will always have to store their data without deleting it. Although this means we have unlimited access to purchase history, it takes up unnecessary space and can make it difficult to manage multiple registries. This could be resolved with the addition of an archive or deletion feature, such that users can create a separation between active and inactive registries once a date has passed for recordkeeping purposes.

5. Queries. What are two common queries likely to be executed against the concept state? (Hint: one is executed by a registry owner, and one by a giver of a gift.)

    - The registry owner will likely want to query which items have been purchased and who purchased them. The giver of a gift will want to query which items have requests in place and the count remaining for them.

6. Hiding purchases. A common feature of gift registries is to allow the recipient to choose not to see purchases so that an element of surprise is retained. How would you augment the concept specification to support this?

    - You could augment the concept specification by adding a ownerVisible Flag to the state, such that it is set at the highest level associated to the registry (where the active flag is also set). Add two actions, which we can call addVisibility and removeVisibility. They would function as follows:

    addVisibility(registry: Registry)
        requires: registry exists, is active, and ownerVisibility is false.
        effects: ownerVisibility flag is set to true, so the recipient can see purchases.

    removeVisibility(registry: Registry)
        requires: registry exists, is active, and ownerVisibility is true.
        effects: ownerVisibility flag is set to false, so the recipient cannot see purchases.

7. Generic types. The User and Item types are specified as generic parameters. The Item type might be populated by SKU codes, for example. Explain why this is preferable to representing items with their names, descriptions, prices, etc.

    - It is preferable to have a unique identifier mapping to an item. Practically, the recipient will most likely have specific preferences for brands and functionalities of an item that may not be captured by simply using a name and price. A generic parameter ensures that we refer to the same exact item and user. In addition, names, descriptions, and prices are subject to change over time. The use of generic types allows for abstraction and consistency across systems.


## Exercise 2: Extending a Familiar Concept
### Questions
1. Complete the definition of the concept state.

**state**

    a set of Users with
        a username String
        a password String

2. Write a requires/effects specification for each of the two actions. (Hints: The register action creates and returns a new user. The authenticate action is primarily a guard, and doesn’t mutate the state.)

**actions**

    register (username: String, password: String): (user: User)
        requires: the username must not already exist in the system
        effects: create a new User with this username and password

    authenticate (username: String, password: String): (user: User)
        requires: there exists a user with the given username and password
        effects: the user is authenticated and will be treated each time as the same user.

3. What essential invariant must hold on the state? How is it preserved?

    All usernames must be unique. It is preserved by the requirement for the register, since the user who is registering must input a username which does not already exist in the system. Therefore, we ensure that all usernames are unique so that there are not multiple passwords mapped to a single username.

4. One widely used extension of this concept requires that registration be confirmed by email. Extend the concept to include this functionality. (Hints: you should add (1) an extra result variable to the register action that returns a secret token that (via a sync) will be emailed to the user; (2) a new confirm action that takes a username and a secret token and completes the registration; (3) whatever additional state is needed to support this behavior.)

    **concept** PasswordAuthentication

    **purpose** limit access to known users

    **principle** after a user registers with a username and password,
    they will confirm their registration by email. 
    They can then authenticate with that same username and password,
    and be treated each time as the same user.

    **state** 

        a set of Users with
            a username String
            a password String
            a set of secretTokens SecretTokens
            a confirmed Flag
        
        a set of SecretTokens with
            a characterString String

    **actions**

    register (username: String, password: String): (user: User, secretToken: SecretToken)

        requires: the username must not already exist in the system

        effects: create a new User with this username and password, returns the user and a secret token that will be emailed to the user

    authenticate (username: String, password: String): (user: User)

        requires: there exists a user with the given username and password

        effects: the user is authenticated and will be treated each time as the same user.

    confirm (user: User, secretToken: SecretToken)

        requires: the user exists, the confirm flag is false, and the user is associated to the given secret token.

        effects: the confirm flag for the given user is set to true, confirming their registration

## Exercise 3: Comparing Concepts

**concept** PersonalAccessToken

**purpose** enable authentication without using a password, access GitHub resources on behalf of yourself

**principle** A user may generate a personal access token with a specified expiration date and scope. The user can then use this token instead of a password to authenticate, then acquiring access to the resources defined in the token's scope. Each token is valid until the expiration date or until it is deleted by the user. A user may create and manage multiple tokens, such that they have various purposes and are deleted once no longer needed.

**state**

    a set of Users with
        a username String
        a password String
        a set of PersonalAccessTokens
        a set of UserPermissions

    a set of PersonalAccessTokens with
        a tokenString String
        an expirationDate Date
        a scope Scope

**actions**

createToken (user: User, scope: Scope, expirationDate: Date): (personalAccessToken: PersonalAccessToken)

    requires: the user exists in the system, expirationDate has not already occurred, scope is a subset of the User's UserPermissions

    effects: generates a personal access token and connects it to the given user, the token only grants permission to the given scope (if none provided, user can only access public info), the token is only valid for user authentication up until the provided expiration date.

deleteToken (user: User, personalAccessToken: PersonalAccessToken)

    requires: the user exists and is associated with the given personal access token

    effects: removes the personal access token association from the given user; the token can no longer be used for authentication

authenticate (username: String, personalAccessToken: PersonalAccessToken)

    requires: the username is tied to a user in the system, the personal access token is tied to that user and it has not expired

    effects: the user is authenticated and granted permission under the given token's scope

PersonalAccessToken differs from PasswordAuthentication due to the extended attributes of a token. Whereas the password will allow users to be treated accordingly, tokens have both an expiration time and an associated scope. The password grants global access, whereas tokens may only provide scoped access. Therefore, there are still security restrictions in place beyond just providing a correct token. Users can also have multiple tokens, but they may only have one unique password. In addition, we allow users to delete tokens but there is no defined behavior for deleting a password. 

The GitHub page should be updated to reflect that the tokens are not exactly like passwords. It is confusing to describe tokens as an alternative to passwords, given that one of the key characteristics for account passwords is that there can only be one per user. There should be more emphasis on the ability to have multiple tokens such that each token can allow for different scopes.

## Exercise 4: Defining Familiar Concepts

### URL Shortener

**concept** URLShortener

**purpose** create short, easily shareable URLs that redirect to longer target URLs

**principle** A user will submit a target URL and can either provide a custom alias or allow the system to automatically generate a shortened URL for them. The short URL will redirect to the target URL when visited. The shortened URLs will have expiration dates, after which the redirect will not work. Users may delete the aliasing before expiration if desired. All aliases are unique across the system.

**state**

    a set of MappedUrls with
        an alias string
        a fullURL string
        an expirationDate Date

**actions**

createLink(alias: String, fullURL: string, expirationDate: Date): (mappedURL: MappedURL)

    requires: fullURL is a valid URL, and alias does not exist already in the system

    effects: if alias is null or an empty string, generate a unique alias between 10 and 30 characters; otherwise, use the provided alias. Create a MappedURL instance with the provided information which will expire on the expiration date, and return this instance.

deleteLink (mappedURL: MappedURL)

    requires: mappedURL exists and has not expired.

    effects: removes the mappedURL from the set of all MappedURLs.

accessLink (alias: String): (fullURL: string)

    requires: alias exists in the system and maps to a fullURL that has not expired

    effects: returns the target URL for the alias

**additional notes** Does not require user login for functionality. Expiration is required. Registered users could later manage multiple links, but not currently required for basic concept.

### Conference Room Booking

**concept** ConferenceRoomBooking

**purpose** manage system for booking conference rooms, accounting for ownership and conflicts

**principle** A user can create a reservation for a specified time, date, and location. The user is the owner of the reservation if they create it, and they may specify additional owners who are granted access to modify or cancel the reservation after it has been created. The reservation can be modified to have a different time/date or space. The owner of the reservation may also update the list of additional owners who can modify the event. If needed, the reservation may also be cancelled.

**state**

    a set of Users with
        a name String
        a set of Reservations
    
    a set of Reservations with
        an eventStart Date
        an eventEnd Date
        an eventName String
        a space String
        an owner User
        an additionalOwners set of Users

**actions**

createBooking (user: User, eventStart: Date, eventEnd: Date, space: String, owners: Users): (reservation: Reservation)

    requires: the user and all additional owners are in the system, eventEnd is after eventStart (both have not occurred yet), and no overlapping reservation exists in the same space

    effects: a reservation is created for the given event times and space. the user and additional owners may modify or cancel this booking. This reservation will be included in the set of reservations for all owners.

modifyBookingDetails (user: User, reservation: Reservation, eventStart: Date, eventEnd: Date, space: String): (reservation: Reservation)

    requires: the user exists and is designated as either the owner or one of the additional owners for the given reservation, eventStart and eventEnd have not occurred yet, and the updated date and space do not overlap with an existing reservation

    effects: update the reservation to have the given event times and space; return the updated Reservation instance.

modifyBookingOwners (user: User, reservation: Reservation, additionalOwners: Users): (reservation: Reservation)

    requires: the user is the owner of the reservation, all additional owners are existing Users

    effects: the set of additionalOwners is updated for the given reservation, which is then returned

cancelReservation (user: User, reservation: Reservation)

    requires: reservation exists in the system and has not already occurred, and the user is an owner or additional owner for the reservation

    effects: reservation is removed from the set all users' Reservations

**additional notes** Assuming that only the original creator of the reservation may modify the list of the additional owners who can modify or cancel a reservation. Users cannot create overlapping reservations in the same event space. Does not account for reoccurring meetings.

### Time-Based One-Time-Password

**concept** TOTP

**purpose** Provide a time-based one-time password for functionality that would support two factor authentication for additional security. Prevents attackers from accessing an account even with stolen passwords.

**principle** A user will register their account with a username and password. After a user attempts to sign in with the correct username and password, the system generates a unique time-limited token and sends it to one of the user's registered devices. Once they input the matching token, they will be authenticated and logged on to the system. Tokens expire automatically after a short period, and multiple devices can be registered for backup access. 

**states**

    a set of Users with
        a username String
        a password String
        a set of Devices
        a set of Tokens
    
    a set of Tokens with
        a tokenString String
        an expirationTime Date

**action**

generateToken (username: String, password: String): (tokenString: String)

    requires: the username and password exist in the system and are tied to the same user

    effects: generates a unique, secure token string that is set to expire in a set amount of time, which is sent to the User's device, and also added to the intended User's set of Tokens.

registerUser (username: String, password: String, device: Device): (user: User)

    requires: username is unique and not already in the system

    effects: creates a User with the provided username and password, links the device to the user, then returns this User

addDevice (user: User, device: Device)

    requires: the user exists and the device is valid

    effects: the device is added to the set of the user's devices

removeDevice (user: User, device: Device)

    requires: the set of devices has at least 2 devices associated at the time this action is called, and the user exists

    effects: the provided device is removed from the set of the user's devices

authenticate (username: String, password: String, tokenString: String)

    requires: the username, password, and tokenString are all linked to the same User, such that the tokenString's associated token has not expired yet

    effects: the user is authenticated

**additional notes** A user may only receive the code on the devices that they have registered in the system. Registering multiple devices supports the use case in which a user loses a device, and thus has a backup. Having an additional code which is linked to a device allows for additional security since an attacker that does not have physical access to the user's device will not be able to acquire this information. However, a user may still be susceptible to a clever attack in which the attacker uses another application (e.g. messages) to ask for the secondary password that was send through the TOTP.
