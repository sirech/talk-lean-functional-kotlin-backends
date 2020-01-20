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

Elevator pitch

- backend services
- using kotlin
- applying functional concepts to make code that is testable, maintainable and easy to understand

---

class: center middle

## what to expect from this talk

???

Content

- code examples that run in production
- introducing some concepts of FP
- not a FP tutorial, not a Kotlin tutorial

---

class: transition

## Mario Fernandez
## Andrei Bechet
 
 **Thought**Works
 
---

class: transition

# Let's start with some context

---

class: center middle

![overview-architecture](images/overview-architecture.png)

???

make it clearer that it is a big ecosystem

---

class: center middle

![overview-architecture-highlight](images/overview-architecture-highlight.png)

---

class: center middle

![backend-architecture](images/backend-architecture.png)

???

- it could be a DB as well

---

class: center middle

## The techs that we will be mentioning

---

class: center middle

.image-grid[
.img[![kotlin](images/kotlin.png)]
.img[![strikt](images/strikt.png)]
.img[![arrow](images/arrow.jpeg)]
.img[![spring-boot](images/spring-boot.png)]
]

---

class: transition

# Our pain points

???

- a picture to show that this was bothering us?

---

class: middle

## I don't know the state of my data

--

## Half of my time I'm just dealing with *null* values

--

## The other half I'm debugging the 500s thrown by our application
 
---

class: transition

# Pain point 1: Uncontrolled change

???

replace with diagram or 3 points with first one highlighted

---

flow that shows uncontrolled change in the middle of the app

---

class: center middle

## Immutability

???

- What is immutability

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

???

- TODO: change to tokens
- Explain we use tokens as the example throughout

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

???

- change to tokens

---

class: center middle

## Dealing with change

---

example of copy constructor

---

class: center middle

```kotlin
expectThat(token) {
    get { name }.isEqualTo("google-oauth2|3234123")
    get { authorities.map { it.authority } }.contains("create:recipes")
}
```

---

benefits of immutability

- easier to reason
- no invalid state

---

class: transition

# Paint point 2: Dealing with null values

---

sample Java code with a ton of if else checks

---

class: middle

## The billion dollar mistake

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

---

class: center middle

## Nullable types

???

- This is what Kotlin offers us out of the box
- TODO: small kotlin logo

---

class: center middle

 ```
Authorization: Bearer bGciOi...JIUzI1NiIs
```

--

```kotlin
fun String.extractToken(): String? = if (startsWith("Bearer"))
    split(" ").last()
else
    null
```

--

```kotlin
header.extractToken()
  ?.let { token -> doStuff(token) }
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
request.getHeader(Headers.AUTHORIZATION)
  ?.let { header ->
    header.extractToken()
      ?.let { jwt ->
        verifier.verify(jwt)
          ?.let { token ->
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
## Option Datatype

???

- TODO: small arrow logo

- as this is the first datatype that we present, this might be the moment to introduce what a datatype is. we can mention that `map` and `flatMap` are the standard functions

---

class: center middle

# Datatype?

.bottom-right[
An FP digression
]

--

class: center middle

## A datatype is an abstraction that encapsulates one reusable coding pattern.

---

class: center middle

```kotlin
interface Operations {
    fun <A, B> DataType<A>.map(f: (A) -> B): DataType<B>
    fun <A, B> DataType<A>.flatMap(f: (A) -> DataType<B>): DataType<B>
}
```

.bottom-right[
An FP digression
]

???

- it is actually a typeclass

---

class: center middle

### hackernoon.com/kotlin-functors-applicatives-and-monads-in-pictures-part-1-3-c47a1b1ce251

---

class: center middle

![option](images/option.png)

???

- It is implemented with sealed classes

---

example with when Some / None ?

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
request.getHeader(Headers.AUTHORIZATION)
  .toOption()
  .flatMap { header ->
    header.extractToken()
      .flatMap { jwt ->
        verifier.verify(jwt)
          .map { token ->
            SecurityContextHolder.getContext().authentication = token
        }
      }
  }
```

???

TODO: proper colors

---

class: center middle

## Not much of an improvement

TODO: sadface

---

class: center middle

## Non-nested syntax thanks to arrow

???

- Similar to await/async in JS

---

class: center middle

```kotlin
Option.fx {
    val (header) = request.getHeader(Headers.AUTHORIZATION).toOption()
    val (jwt) = header.extractToken()
    val (token) = verifier.verify(jwt)
    SecurityContextHolder.getContext().authentication = token
}
```

???

TODO: align the code with the previous example

---

class: center middle

```kotlin
@Test
fun `verify does not work with a invalid jwt token`() {
    expectThat(
      RemoteVerifier(keySet).verify(jwt)
*   ).isEmpty()
}
```

???

TODO: replace with code without customer assertion

---

- deal with null!

TODO: conclusion

---

class: transition

# Pain 3: Exceptions

---

diagram with a typical stackstrace

---

class: center middle

## Verifying our token

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

???

- TODO: adapt code to avoid mentioning jwt

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
public DecodedJWT verifyByCallingExternalApi(String token);
```

???

- TODO: Make it clearer that this is the implementation

---

class: center middle

![exception](images/exception.png)

???

- verify will throw an exception whenever it is not succesful
- TODO: to which diagram should this refer to?

---

class: center middle

## Exceptions make the flow implicit

???

- the error case will ignore the path we defined before

---

class: center middle

## Exceptions force you to be aware of the internal implementation

.bottom-right[
kotlinlang.org/docs/reference/exceptions.html
]

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

## Either DataType

---

class: center middle

## Option and Either are quite similar

.bottom-right[
An FP digression
]

???

- TODO: mention that Option and Either are similar? `Kind` perhaps?

---

class: center middle

```kotlin
interface Operations {
    fun <A, B> DataType<A>.map(f: (A) -> B): DataType<B>
    fun <A, B> DataType<A>.flatMap(f: (A) -> DataType<B>): DataType<B>
}
```

.bottom-right[
An FP digression
]

???

- TODO correct types

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
- TODO: 

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
*          .mapLeft { DownstreamException("Unable to fetch products") }
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

???

TODO: maybe start with this before the going into Either

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

???

TODO: conclusion

---

remember slide with the 3 pains from the beginning

---

class: transition

# Bonus

---

class: center middle

## Purely functional code

---

diagram of edge of the world

---

class: center middle

## IO

???

Next steps slide (maybe) ? 

---

class: center middle

`IO<Either<JWTVerificationException, TokenAuthentication>>`

---

class: center middle

## We are hitting the limit of what's convenient to do with Kotlin and Arrow here

---

class: transition

# Conclusions

???

- TODO: 

---

???

TODO: journey metaphor

---

class: center middle

# Incremental adoption

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









