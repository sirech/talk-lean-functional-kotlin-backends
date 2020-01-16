title: Lean backends using functional Kotlin
class: animation-fade
layout: true


<!-- This slide will serve as the base layout for all your slides -->

---

class: impact full-width
background-image: url(images/background1.jpg)

.impact-wrapper[
# {{title}}
]

???

Rough time distribution

- Intro (who we are / goal of the talk / agenda) 5
- Functional programming 7
- Where do we use this? 3
- Immutability 5
- Null Safety 10
- Exceptions  10 
- Side Effects 5
- Wrap Up (conclusion / thank you) 5

---

class: transition

## Mario Fernandez
## Andrei Bechet
 
 **Thought**Works
 
---

class: center middle

## What to expect from this talk

???

- introduce functional concepts
- show code that we use in day to day and help us
 
---

class: transition

# Agenda

---

class: center middle

- Functional programming
- Where do we use this?
- Immutability
- Null Safety
- Exceptions
- Side Effects

---

class: transition

# Functional Programming

???

Why? 
- packing a lot of action into fewer lines of code
  - simplicity => we are dealing only with the local scope
  - elegant => separation between symbolic logic vs. algorithms 
  - easier to test and to reason about it

How? 
- FP, by nature, describes the *logic* rather than the control flow
- becoms closer to the domain since we create privites as we go
- declarative 

- more on the next slides while presenting kotlin in the backend

---

class: center middle

![FP components](images/fp-components.jpeg)

???

- just mention the components generically speaking
- tell that we will cover more of them as we proceed

---

class: center middle

## Pure functions and Immutability

???

Functions:
- build expressions which take arguments and produce new values
  - unlike in imperative programming where they are conceptualized as side-effecting behaviors
- what makes them pure? 
  - same returned value for the same arguments
  - have no side-effects (like mutation...)

Immutability
- simplicity (easy to reason about it since it's predictable)
- makes it easy for scaling and fixes part of side-effects 

Link to next slide:
- FP, by nature, describes the *logic* rather than the control flow
- leads to becoming close to the domain => create privites as we go

---

class: center middle

## Higher order functions

???

- functions that take other functions as input parameters
- functions that operate on other functions
- how do we use them? 
  - they allow us to abstract 
  - and compose actions

---

class: center middle

## Focus on What instead of How
Separate the *domain* from the *process of execution*

???

Declarative 
- focus on the phenomena and the high-level behavior
FP allows to address them adequately and separately
- the execution becomes just the computation of some values
In FP we basically patterns become primitives
- abstracting away side-effects 

FP is all about removing the side-effects and mutations ... basically _removing_ the time from the equation

---

class: center middle

## Data Types and Type Classes

???

- data types = abstractions of our domain (leads to "new" primitives) 
- data types DO NOT contain the logic
- type classes = describe abstract behaviors that affect multiple data types
- concrete instances targeting specific data types have to be defined 
- type classes basically provide polymorphic declarations 

- basically whenever we need ot do something with the data types we inject the behavior using the type classes

---

class: center middle

![kotlin](images/kotlin.png)

???

- functions are first class => higher order functions
- immutability 
- makes null explicit

- write concise and expressive code
- scales with the help of couroutines
- has nice language features 
- kotlin can be adopted incrementaly (given JVM)


---

class: transition

# Where do we use this?
## Lean Backends

---

class: center middle

![backend-architecture](images/backend-architecture.png)

---

class: center middle

## Test Driven

---

class: transition

# Immutable data

---

class: center middle

```kotlin
data class User(
        val id: Id,
        val firstName: FirstName,
        val lastName: LastName,
        val phoneNumber: String,
        val dateOfBirth: DateOfBirth?,
        val location: City?
)
```

---

collection example

---

class: center middle

## JSON

---

class: center middle

```kotlin
@JsonIgnoreProperties(ignoreUnknown = true)
data class User(
        @JsonAlias("BusinessPartnerId")
        val id: Id,
        @JsonAlias("NameFirst")
        val firstName: FirstName,
        @JsonAlias("NameLast")
        val lastName: LastName,

        @JsonAlias("BpData_To_BpAddress")
        @JsonDeserialize(converter = AddressListSanitizer::class)
        val addresses: ListOfResults<Address> = ListOfResults.empty()
)
```

---

TODO: mapping data to another?

---

TODO: lenses?

---

class: center middle

![strikt](images/strikt.png)

---

class: center middle

```kotlin
expectThat(token) {
    get { name }.isEqualTo("google-oauth2|3234123")
    get { authorities.map { it.authority } }.contains("create:recipes")
}
```

---

class: center middle

```console
org.opentest4j.AssertionFailedError: 
▼ Expect that Some(TokenAuthentication@52789c41):
  ▼ TokenAuthentication@52789c41: 
    Authenticated: true; 
    Authorities: profile, create:recipes:
    ▼ name:
      ✗ is equal to "google-oauth2|3234123" : found "google-oauth2|dude"
```

---

class: transition

# Null Safety

---

class: middle

## The billion dollar mistake

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

---

class: center middle

## Decoding JWT tokens

---

class: center middle

.bottom-right[
### jwt.io/
]

```shell
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ
.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

---

class: center middle

```json
{"alg":"HS256","typ":"JWT"}
```

```json
{"sub":"1234567890","name":"John Doe","iat":1516239022}
```

```
signature
```

---

class: center middle

## Nullable types

---

class: center middle

```kotlin
fun String.extractToken(): String? = if (startsWith("Bearer"))
    split(" ").last()
else
    null
```

--

```kotlin
header.extractToken()?.let { token -> 
    doStuff(token)
}
```

---

class: center middle

## It can get messy

---

class: center middle

![jwt-flow](images/jwt-flow.png)

---

class: center middle

```kotlin
request.getHeader(Headers.AUTHORIZATION)?.let { header ->
    header.extractToken()?.let { jwt ->
        verifier.verify(jwt)?.let { token ->
            SecurityContextHolder.getContext().authentication = token       
        }
    }
}
```

---
class: center middle

![callback-hell](images/callback-hell.jpeg)

---

class: center middle
## Option

---

class: center middle

![option](images/option.png)

---

class: center middle

```kotlin
fun String.extractToken(): String? = if (startsWith("Bearer"))
    split(" ").last()
else
    null
```

```kotlin
fun String.extractToken(): Option<String> = startsWith("Bearer ")
        .maybe { split(" ").last() }
```

---

class: center middle

## Let's try our previous example with *Option*

---

class: center middle

```kotlin
request.getHeader(Headers.AUTHORIZATION).toOption().flatMap { header ->
    header.extractToken().flatMap { jwt ->
        verifier.verify(jwt).toOption().map { token ->
            SecurityContextHolder.getContext().authentication = token
        }
    }
}
```

---

class: center middle

## Not much of an improvement

---

class: center middle

## Monadic comprehensions

---

class: center middle

```kotlin
Option.fx {
    val (header) = request.getHeader(Headers.AUTHORIZATION).toOption()
    val (jwt) = header.extractToken()
    val (token) = verifier.verify(jwt).toOption()
    SecurityContextHolder.getContext().authentication = token
}
```

---

class: center middle

## Custom assertion for testing

---

class: center middle

```kotlin
inline fun <reified T> Assertion.Builder<Option<T>>.isEmpty() =
        isA<None>()
```

---

class: center middle

```kotlin
@Test
fun `verify does not work with a invalid jwt token`() {
    expectThat(RemoteVerifier(keySet)
      .verify(jwt)
      .toOption()
    ).isEmpty()
}
```

---

class: transition

# Exceptions

---

class: center middle

## Verifying JWT Tokens

---

class: center middle

![jwt-flow-highlight](images/jwt-flow-highlight.png)

---

class: center middle

```kotlin
interface Verifier {
    /**
     * @param jwt a jwt token
     * @return whether the token is valid or not
     */
    fun verify(jwt: String): TokenAuthentication
}
```

---

class: center middle

## That signature is not quite telling the truth

---

class: center middle

```java
/**
 * Perform the verification against the given Token
 *
 * @param token to verify.
 * @return a verified and decoded JWT.
 * @throws AlgorithmMismatchException     
 * @throws SignatureVerificationException 
 * @throws TokenExpiredException          
 * @throws InvalidClaimException          
 */
@Override
public DecodedJWT verify(String token) 
  throws JWTVerificationException {
    DecodedJWT jwt = new JWTDecoder(parser, token);
    return verify(jwt);
}
```

???

- we are using auth0's library to decode a token

---

class: center middle

![exception](images/exception.png)

???

- verify will throw an exception whenever it is not succesful

---

class: center middle

## Exceptions make the flow implicit

???

- the error case will ignore the path we defined before

---

class: center middle

## Exceptions force you to be aware of the internal implementation

???

- you cannot know that unless you inspect the implementation

---

class: center middle

```kotlin
@ExceptionHandler(JWTVerificationException::class)
fun handleException(exception: JWTVerificationException):
  ResponseEntity<ErrorMessage> {
    return ResponseEntity
      .status(HttpStatus.BAD_GATEWAY)
      .body(ErrorMessage.fromException(exception))
}
```

---

class: center middle

## Either

---

class: center middle

![either](images/either.png)

???

- Either is a disjoint union. It means the result can be either one or the other, but not both

---

class: center middle

```kotlin
interface Verifier {
    /**
     * @param jwt a jwt token
     * @return whether the token is valid or not
     */
    fun verify(jwt: String): 
      Either<JWTVerificationException, TokenAuthentication>
}
```

---

class: center middle

## Isolating the problematic code

---

class: center middle

```kotlin
private fun JWTVerifier.unsafeVerify(jwt: String) = try {
    verify(jwt).right()
} catch (e: JWTVerificationException) {
    e.left()
}
```

???

- There is a `Try` datatype for this, but it has been deprecated
- This is technically a side effect, which we will get back to later

---

class: center middle

## Operating with Either

---

class: center middle

```kotlin
override fun verify(jwt: String)
       : Either<JWTVerificationException, TokenAuthentication> {
    val key = key(keySet)
    val algorithm = algorithm(key)
    val verifier = verifier(algorithm, leeway)
    return verifier
*           .unsafeVerify(jwt)
*           .map { it.asToken() }
}
```

---

class: center middle

```kotlin
Either.fx<DownstreamException, List<Product>> {
    // Either<Throwable, ResponseEntity<UnprocessedResponse>> 
    val response = unsafeRequest() 
    val (body) = response
           .mapLeft { DownstreamException("Unable to fetch products") }
    body.map()
}
```

---

class: center middle

```kotlin
@GetMapping("{id}")
fun recipe(@PathVariable id: Int): ResponseEntity<RecipeDetails> {
    return when (val result = repository.find(id)) {
        is Either.Left -> ResponseEntity.status(result.a).build()
        is Either.Right -> ResponseEntity.ok(result.b)
    }
}
```

---

class: center middle

## Result
### kotlin-stdlib

---

class: center middle


```kotlin
fun unsafeOp() =
        runCatching { 
            doStuff()
        }.getOrElse { exception -> handle(exception) }
```

---

class: center middle

## Moar custom assertions

---

class: center middle

```kotlin
inline fun <reified T, reified U> Assertion.Builder<Either<U, T>>
  .isRight() =
        isA<Either.Right<T>>()
                .get { b }
```

---

class: center middle

```kotlin
@Test
fun `verify works if the expiration is not taken into account`() {
    val hundredYears = 3600L * 24 * 365 * 100
    val verifier = RemoteVerifier(keySet, hundredYears)

    expectThat(verifier.verify(jwt)).isRight().and {
        get { name }
          .isEqualTo("google-oauth2|111460419457288935787")
        get { authorities.map { it.authority } }
          .contains("create:recipes")
    }
}
```

---

class: transition

# Side Effects

---

class: center middle

## Purely functional code

---

IO

---

IO<Either<E, T>> type

---

the difficulties of mapping IO to a typed exception

---

class: center middle

# This is but a journey

---

diagram with different stages

---

class: transition

# Conclusions

???


---

class: center middle

# Kotlin is a versatile language

???

- kotlin is a versatile language
- kotlin can be successfully used for a lot more things than just android apps
  - like backends

---

class: center middle

# FP != only monads 

(or other fancy keywords)

???

- FP does require us to think a bit differently but it is not only about monads and facy keywords

---

class: center middle

# FP can improve our codebase

???

- functional style becomes helpful for multiple reasons
  - simplicity
  - elegance
  - testability and easy to reason about
  - scalability
  - safety

---

class: center middle

# Adopt functional style Kotlin step by step

???

---

class: impact full-width
background-image: url(images/background5.jpg)

.impact-wrapper[
## JOIN OUR COMMUNITY

<br />

### *26 years* experience
### *42 offices* in 13 countries
### *Thought leader* in agile software development and continuous delivery
### *6000+* thoughtworkers worldwide
### *300+* thoughtworkers in Germany

<br />

#### de-recruiting@thoughtworks.com
]

---








