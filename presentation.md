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

---

class: transition

## Mario Fernandez
## Andrei Bechet
 
 **Thought**Works
 
---

class: transition

# Agenda

---

class: transition

# Functional Programming

---

class: center middle

## Functions

---

class: center middle

## Higher order functions

---

class: center middle

## Testing functions

---

class: center middle

## New operators

???

- map
- flatMap

---

class: transition

# Use case: Microservices

---

pic of target architecture

---

class: transition

# Immutable data

---

simple data class

---

parsing json data

---

class: transition

# Null Safety

---

class: middle

# The billion dollar mistake

> I call it my billion-dollar mistake. It was the invention of the null reference in 1965. At that time, I was designing the first comprehensive type system for references in an object oriented language (ALGOL W). My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn't resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

---

class: center middle

## Decoding JWT tokens

---

context about JWT

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

# Nullable types

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
val header = "Bearer eJ..."
header.extractToken()?.let { token -> 
    doStuff(token)
}
```

---

class: center middle

# It can get messy

---

picture of flow

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

pic of callback hell

---

class: center middle
# Option

---

what is an option

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

# Let's try our previous example with *Option*

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

# Monadic comprehensions

---

context

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

class: transition

# Exceptions

---

class: center middle

# Verifying JWT Tokens

---

diagram verification flow

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

# Representing failure

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

---

diagram of server throwing

---

class: center middle

# Exceptions make the flow implicit

---

class: center middle

```kotlin
@ExceptionHandler(DownstreamException::class)
fun handleException(exception: DownstreamException) =
        ResponseEntity
                .status(HttpStatus.BAD_GATEWAY)
                .body(ErrorMessage.fromException(exception))
```

---

class: center middle

# Result
## kotlin-stdlib

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

# Either

---

what is an either

---

class: center middle

```kotlin
override fun verify(jwt: String)
*       : Either<JWTVerificationException, TokenAuthentication> {
    val key = key(keySet)
    val algorithm = algorithm(key)
    val verifier = verifier(algorithm, leeway)
    return verifier
            .unsafeVerify(jwt)
            .map { it.asToken() }
}
```

--

```kotlin
private fun JWTVerifier.unsafeVerify(jwt: String) = try {
    verify(jwt).right()
} catch (e: JWTVerificationException) {
    e.left()
}
```

---

class: center middle

```kotlin
Either.fx<DownstreamException, List<Product>> {
    // Either<Throwable, ResponseEntity<UnprocessedResponse>> 
    val response = unsafeRequest() 
    val (withError) = response
           .mapLeft { DownstreamException("Unable to fetch products") }
    val (body) = withError.nonThrowingEmptyBody
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

# Side Effects

---

class: center middle

# Purely functional code

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








