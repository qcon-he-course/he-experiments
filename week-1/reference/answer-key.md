# Week 1 answer key: the six questions

Use it to score your own runs. Keep it out of the seeded repository: an agent that can read the answers isn't answering the questions. `run.sh` clones PetClinic outside this repository, so the agent in the container never sees this file.

All six answers are for the seeded classroom copy of PetClinic, `github.com/qcon-he-course/he-petclinic` at tag `week-1` (commit `b107a86`). Paths are relative to that repository unless they say otherwise. Two checks back every answer:

- **Verified in code:** every line reference was read on `b107a86` on 19 October 2026 and says what the answer claims. Where the behaviour depends on a framework, the framework's source was read too.
- **Verified in the running app:** on 18 October 2026, seeded PetClinic at `b107a86` was started with `./mvnw spring-boot:run` and no options (Java 21, in the `he-ladder` image), and each answer was checked against it. The running app agreed with all six answers.

Our stored runs (`../results/summary.md`) were marked against this key by a model, Claude Fable 5.1, and we confirmed the marks by hand.

**Marking.** An answer is right when it gives the outcome and the reason that decides it. "H2" alone is half right. "H2, because `database=h2` and no datasource URL is set" is right. Question 6 scores one point per step, out of 6.

The questions, as every run asks them (`../experiments/questions.txt`):

```text
1. What is this project about? How do I get started?
2. When you start PetClinic with no options, which database does it use, and what decides that?
3. How long does PetClinic keep the vet list in its cache?
4. Jean Coleman already has a pet called Max. Can she add another pet called "max"?
5. You search owners for "davis", all lower case. What do you see?
6. Trace what happens when an owner books a visit for a pet, from submitting the form to the row in the database. List each step in order, with the class and method involved.
```

## 1. What the project is and how to start it

**Answer.** Spring PetClinic, Spring's sample application. It is a web application for a vet clinic's owners, pets, visits and vets, built with Spring Boot, Spring MVC, Thymeleaf and Spring Data JPA. To start it, run `./mvnw spring-boot:run` (or `./gradlew bootRun`) with Java 17 or later, then open http://localhost:8080. There is no database to set up: with no options it starts an embedded H2 database (question 2).

**Evidence** (verified in code):

| Where | What it shows |
|---|---|
| Running app, 18 October | `./mvnw spring-boot:run` starts Tomcat on port 8080 with no database to start first |
| `README.md:17` | "Java 17 or later is required for the build, and the application can run with Java 17 or newer." |
| `README.md:28`, `:33` | `./mvnw spring-boot:run` and `./gradlew bootRun` |
| `README.md:36` | "You can then access the Petclinic at <http://localhost:8080/>." |
| `pom.xml:19` | `<java.version>17</java.version>` |
| `src/main/resources/application.properties:2` | `database=h2` |
| `pom.xml:163-164` | `spring-boot-docker-compose` is test scope only, so a normal run starts no containers |

**Rigged README line.** `README.md:71-72`: "In its default configuration, Petclinic uses PostgreSQL. Start the database with `docker compose up postgres` before running the app".

**Typical wrong answer.** Start PostgreSQL first with `docker compose up postgres`, taken from the rigged line. The right answer needs the agent to check that claim against the configuration.

## 2. Which database it uses with no options

**Answer.** H2, in memory. `database=h2` in `application.properties` selects the H2 schema and data scripts. No datasource URL is set, so Spring Boot starts an embedded H2 database from the H2 dependency. MySQL or PostgreSQL are used only with the `mysql` or `postgres` profile, which sets `database` and a datasource URL.

**Evidence** (verified in code):

| Where | What it shows |
|---|---|
| Running app, 18 October | no active profile; Hikari's first connection is `jdbc:h2:mem:<uuid>` |
| `src/main/resources/application.properties:2-4` | `database=h2`, and the schema and data locations built from `${database}` |
| `src/main/resources/application.properties` | no `spring.datasource.*` property; only the profile files set one |
| `src/main/resources/application-postgres.properties:2-3` | `database=postgres` and `spring.datasource.url=${POSTGRES_URL:jdbc:postgresql://localhost/petclinic}` |
| `src/main/resources/application-mysql.properties:3` | the MySQL datasource URL, under the `mysql` profile |
| `pom.xml:77-79` | `com.h2database:h2`, runtime scope |

**Rigged README line.** `README.md:71`: "In its default configuration, Petclinic uses PostgreSQL."

**Typical wrong answer.** PostgreSQL, from the rigged line.

## 3. How long the vet list stays cached

**Answer.** Until the application restarts. Spring Boot runs a plain Caffeine cache manager with Caffeine's defaults, which set no expiry and no size limit. Nothing in the code evicts the cache. So the first vet list read from the database is served from then on. The one other way to clear it is the actuator's caches endpoint (`DELETE /actuator/caches`), which is open because `application.properties` exposes every actuator endpoint. Only the vets are cached.

The pom has the JCache API and plain Caffeine, but no JCache provider. Spring Boot sets up JCache only when it finds a provider, so it falls through to its Caffeine configuration. `CacheConfiguration` still matters: its `@EnableCaching` turns caching on. Its `JCacheManagerCustomizer` and `MutableConfiguration` are never applied.

**Evidence:**

| Where | What it shows | How checked |
|---|---|---|
| Running app, 18 October | the `cacheManager` bean is `CaffeineCacheManager`; after `GET /vets.html`, `/actuator/caches` shows `vets` as Caffeine's `UnboundedLocalCache`: no size limit and no expiry | Verified in the running app |
| `src/main/java/org/springframework/samples/petclinic/vet/VetRepository.java:45`, `:55` | `@Cacheable("vets")` on both `findAll` methods | Verified in code |
| `pom.xml:69` | `javax.cache:cache-api`, the JCache API only | Verified in code |
| `pom.xml:83` | `com.github.ben-manes.caffeine:caffeine`, without Caffeine's separate `jcache` provider artifact | Verified in code |
| `pom.xml` dependency list | no other dependency is a JCache provider (no Ehcache, Hazelcast, Infinispan or Caffeine `jcache`) | Verified in code (direct dependencies only) |
| `pom.xml` parent | `spring-boot-starter-parent` 4.1.0 | Verified in code |
| Spring Boot `v4.1.0`, `module/spring-boot-cache/.../JCacheCacheConfiguration.java:60-62`, `:158-170` | JCache is set up only if `spring.cache.jcache.provider` is set or exactly one `CachingProvider` is found; with none, the condition fails ("did not find JSR-107 provider") | Verified in framework source |
| Spring Boot `v4.1.0`, `.../CaffeineCacheConfiguration.java:43-46` | the Caffeine cache manager applies when Caffeine is on the classpath and no other cache manager exists | Verified in framework source |
| `src/main/java/org/springframework/samples/petclinic/system/CacheConfiguration.java:32` | `@EnableCaching` | Verified in code |
| `CacheConfiguration.java:37`, `:50` | the `JCacheManagerCustomizer` that creates "vets" with `new MutableConfiguration<>().setStatisticsEnabled(true)`: the bean exists but is never applied | Verified in the running app |
| `src/` and `pom.xml` | no `@CacheEvict`, `@CachePut` or `spring.cache.*` setting | Verified in code (grep) |
| `src/main/resources/application.properties:21` | `management.endpoints.web.exposure.include=*` | Verified in code |
| `pom.xml:44` | the actuator starter | Verified in code |

**Marking.** The rule we marked by: the answer is right only if its own explanation is Spring Boot's Caffeine cache manager, because the pom has no JCache provider. An answer that states `CacheConfiguration`'s `MutableConfiguration` or a JCache default as the reason, as fact, is partial: "right lifetime, mixed reason". Hedging the right reason, with no wrong reason stated as fact, is still right.

**Rigged README line.** `README.md:160`, the "Caching" row: "the vet list is refreshed from the database every 10 minutes".

**Typical wrong answers.** Ten minutes, from the rigged line. Or until restart, because `CacheConfiguration`'s `MutableConfiguration` sets no expiry policy: the right lifetime for the wrong reason.

## 4. A second pet called "max"

**Answer.** No. The form shows "is already in use" next to the name. The controller compares the new name with the owner's existing pets, ignoring case. The database enforces the same rule: a unique constraint on owner and name, where the name column ignores case. The controller's check rejects the form first, so the constraint is a second line of defence. A different owner can have a pet called Max: two owners already have a Lucky.

**Evidence** (verified in code):

| Where | What it shows |
|---|---|
| Running app, 18 October | `GET /owners?lastName=Coleman` redirects to owner 6; posting a new pet named `max` returns the form with "is already in use" on the name field |
| `src/main/resources/db/h2/data.sql:30` | Jean Coleman, the sixth owner |
| `src/main/resources/db/h2/data.sql:43` | her pet Max (`owner_id` 6) |
| `src/main/resources/db/h2/data.sql:44`, `:47` | Lucky for owners 7 and 10 |
| `src/main/java/org/springframework/samples/petclinic/owner/PetController.java:111-112` | on a new pet, `owner.getPet(pet.getName(), true) != null` rejects the name with code `duplicate` |
| `src/main/java/org/springframework/samples/petclinic/owner/Owner.java:147-150` | `getPet(name, ignoreNew)` compares with `equalsIgnoreCase` |
| `src/main/resources/db/h2/schema.sql:48` | `name VARCHAR_IGNORECASE(30)` on `pets` |
| `src/main/resources/db/h2/schema.sql:55` | `unique_owner_pet_name UNIQUE (owner_id, name)` |
| `src/main/resources/messages/messages.properties:4` | `duplicate=is already in use` |
| `src/main/resources/messages/messages_en.properties` | empty, so an English browser falls back to `messages.properties`; other locales show their own translation |

**Rigged README line.** `README.md:66`: "Pet names are case-sensitive, so `Max` and `max` are two different pets."

**Typical wrong answer.** Yes, because pet names are case-sensitive, from the rigged line.

## 5. Searching owners for "davis"

**Answer.** A list with both Davises, Betty and Harold. The repository query is a case-sensitive prefix match, `findByLastNameStartingWith`. It finds them only because H2's schema declares `last_name` as `VARCHAR_IGNORECASE`, and H2 compares `LIKE` on such a column ignoring case. On PostgreSQL, where the column is `TEXT`, the same search shows "has not been found". The list shows five owners per page. Exactly one match would go straight to that owner's page.

**Evidence:**

| Where | What it shows | How checked |
|---|---|---|
| Running app, 18 October | the owners list, not a redirect, with Betty Davis and Harold Davis | Verified in the running app |
| `src/main/java/org/springframework/samples/petclinic/owner/OwnerRepository.java:45` | `findByLastNameStartingWith(String lastName, Pageable pageable)`, with no `IgnoreCase` | Verified in code |
| `src/main/resources/db/h2/schema.sql:39` | `last_name VARCHAR_IGNORECASE(30)` on `owners` | Verified in code |
| `src/main/resources/db/h2/data.sql:26`, `:28` | Betty Davis and Harold Davis | Verified in code |
| `src/main/java/org/springframework/samples/petclinic/owner/OwnerController.java:116`, `:122`, `:142` | no match shows the find form with an error; one match redirects to the owner; page size 5 | Verified in code |
| `src/main/resources/messages/messages.properties:3` | `notFound=has not been found` | Verified in code |
| `src/main/resources/db/postgres/schema.sql:29` | `last_name TEXT` under the `postgres` profile | Verified in code |
| H2 `h2/src/main/org/h2/expression/condition/CompareLike.java:151-152` | `LIKE` sets `ignoreCase` when the left side is `VARCHAR_IGNORECASE` | Verified in framework source, on H2's main branch; the exact H2 version PetClinic resolves was not checked |

**Rigged README line.** `README.md:67`: "Owner search is case-sensitive: type the last name as it was entered, for example `Davis`."

**Typical wrong answer.** Nothing found, because the search is case-sensitive, from the rigged line. An agent that reads only the Java code reaches the same wrong answer.

## 6. Booking a visit, from form to database row

**Answer.** Six steps. Steps 2 and 5 are done by the framework, through an annotation and a cascade mapping.

| Step | What happens | Evidence |
|---|---|---|
| 1 | The visit form posts a date and a description to `/owners/{ownerId}/pets/{petId}/visits/new`. The form has no `action`, so it posts to the URL it was loaded from. | `src/main/resources/templates/pets/createOrUpdateVisitForm.html:30-34` |
| 2 | Before the handler runs, Spring MVC calls `VisitController.loadPetWithVisit`, a `@ModelAttribute("visit")` method. It loads the owner and the pet, puts both in the model, creates the new `Visit` and adds it to the pet. Nothing in the code calls this method directly. | `VisitController.java:63-80`; `pet.addVisit(visit)` at `:79` |
| 3 | `VisitController.processNewVisitForm` validates. The description is required, and the date must be after today: today itself is rejected. Errors show the form again. A new `Visit` defaults to tomorrow's date. | `VisitController.java:97-106`, date check at `:100-102`; `Visit.java:43` (`@NotBlank`); `Visit.java:50-51` |
| 4 | `owner.addVisit(petId, visit)` finds the pet and adds the visit again. Step 2 already did, and the pet's visits are a `LinkedHashSet` of the same object, so nothing changes. An agent that says so gets the point. | `VisitController.java:108`; `Owner.java:176-186`; `Pet.java:59`, `:81-83` |
| 5 | `owners.save(owner)` saves the owner. JPA cascades from owner to pets to visits, which inserts the row into `visits`, with `pet_id` from the join column. | `VisitController.java:109`; `Owner.java:67` (`cascade = CascadeType.ALL`); `Pet.java:56-57` (`CascadeType.ALL`, `@JoinColumn(name = "pet_id")`); `src/main/resources/db/h2/schema.sql:58-64` |
| 6 | The browser is redirected to `/owners/{ownerId}` with the flash message "Your visit has been booked". | `VisitController.java:110-111` |

**How checked.** Verified in code. In the running app on 18 October, posting a visit for owner 1, pet 1 redirected to `/owners/1` with "Your visit has been booked", and the owner's page, loaded again from the database, listed the visit with its date and description. Step 4's "nothing changes" also rests on `BaseEntity` not overriding `equals`, which was checked by grep in `src/main/java/org/springframework/samples/petclinic/model/`.

**Why this question is here.** Call graphs and language servers don't show who calls `loadPetWithVisit` or how the row reaches the table. An agent that relies only on those tools misses steps 2 and 5. Experiments 3 and 4 can show this.

**Typical wrong answers.** A service layer between controller and repository: PetClinic has none. An explicit `VisitRepository` call: there is no `VisitRepository`, and the save goes through the owner (`owner/` has `OwnerRepository` and `PetTypeRepository` only). A vet chosen on the form: the form has a date and a description only.

## Check the evidence yourself

Run these in a clone of PetClinic at `b107a86`, outside this repository. Each should print the line cited above.

```bash
P=src/main/java/org/springframework/samples/petclinic
grep -n "Java 17\|mvnw spring-boot:run\|gradlew bootRun\|localhost:8080" README.md
grep -n "In its default configuration\|case-sensitive\|10 minutes" README.md
grep -n "^database=\|exposure.include\|datasource" src/main/resources/application.properties   # no datasource line
grep -n "cache-api\|<artifactId>caffeine\|<artifactId>h2\|spring-boot-docker-compose\|starter-parent" -A1 pom.xml
grep -n "Cacheable" $P/vet/VetRepository.java
grep -n "EnableCaching\|createCache\|MutableConfiguration<>" $P/system/CacheConfiguration.java
grep -rn "CacheEvict\|CachePut\|spring.cache" src pom.xml          # expect nothing
grep -n "getPet(pet.getName(), true)\|\"duplicate\"" $P/owner/PetController.java
grep -n "equalsIgnoreCase" $P/owner/Owner.java
grep -n "VARCHAR_IGNORECASE\|unique_owner_pet_name" src/main/resources/db/h2/schema.sql
grep -n "Coleman\|'Max'\|'Lucky'\|Davis" src/main/resources/db/h2/data.sql
grep -n "duplicate=\|notFound=" src/main/resources/messages/messages.properties
grep -n "findByLastNameStartingWith" $P/owner/OwnerRepository.java
grep -n "isEmpty()\|getTotalElements() == 1\|pageSize = " $P/owner/OwnerController.java
grep -n "last_name" src/main/resources/db/postgres/schema.sql
grep -n "ModelAttribute(\"visit\")\|pet.addVisit\|isAfter\|addVisit(petId\|owners.save\|booked\|redirect" $P/owner/VisitController.java
grep -n "cascade\|JoinColumn\|LinkedHashSet" $P/owner/Owner.java $P/owner/Pet.java
grep -n -A6 "CREATE TABLE visits" src/main/resources/db/h2/schema.sql
```
