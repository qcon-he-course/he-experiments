# Architecture map: evidence

Source: `.harness/diagrams/architecture.json` → `.harness/diagrams/architecture.html`
(commit `bc2e804`; archify `deliver --quality showcase`: 9/9 checks, 0 errors, 0 warnings).

Paths below are relative to `src/main/java/org/springframework/samples/petclinic/` unless they start with `src/`, `pom.xml` or `.harness/`.
"cgr" means a `cgr graph` query run in this session; `callers X` lists the call sites returned.

## Nodes

| Node id | Label | CodeBoarding component | Package / classes | Source | Status |
|---|---|---|---|---|---|
| `browser` | Browser | — (external) | — | The controllers return Thymeleaf view names, e.g. `owner/OwnerController.java:91,138`, `vet/VetController.java:62`; `src/main/resources/application.properties:7` | VERIFIED (that the app serves HTML views); the browser is the implied client |
| `system` | System Infrastructure | `3` System Infrastructure | `system`: WelcomeController, CrashController, WebConfiguration (CacheConfiguration listed by CodeBoarding, see notes) | `system/WelcomeController.java:25`, `system/CrashController.java:31`, `system/WebConfiguration.java:44-58` | VERIFIED |
| `owner-web` | Owner & Pet Web | `1.1` Owner & Pet Web Management (controllers only) | `owner`: OwnerController, PetController, PetValidator | `owner/OwnerController.java:48-55`, `owner/PetController.java:47-57,94-96` | VERIFIED |
| `visit-web` | Visit Scheduling | `1.3` Visit Scheduling (controller only) | `owner`: VisitController | `owner/VisitController.java:42-47` | VERIFIED |
| `vet` | Veterinarian Mgmt | `2` Veterinarian Management | `vet`: VetController, VetRepository, Vet, Specialty, Vets | `vet/VetController.java:36-41`, `vet/VetRepository.java:38` | VERIFIED |
| `pettype` | Pet Type Lookup | `1.2` Pet Type Lookup | `owner`: PetTypeFormatter (`@Component`), PetTypeRepository | `owner/PetTypeFormatter.java:36-43`, `owner/PetTypeRepository.java:30` | VERIFIED |
| `owner-repo` | OwnerRepository | part of `1.1` (split out here) | `owner`: OwnerRepository `extends JpaRepository<Owner, Integer>` | `owner/OwnerRepository.java:36` | VERIFIED |
| `owner-agg` | Owner Aggregate | parts of `1.1` (Owner, PetType), `1.3` (Visit), `1.4` (Pet) | `owner`: Owner, Pet, Visit, PetType entities | `owner/Owner.java:48-50,67-70`, `owner/Pet.java:44-59`, `owner/Visit.java:35-37` | VERIFIED |
| `base` | Base Domain Model | `4` Base Domain Model | `model`: BaseEntity, NamedEntity, Person (`@MappedSuperclass`) | `model/BaseEntity.java:32`, `model/NamedEntity.java:30-31`, `model/Person.java:28-29` | VERIFIED |
| `db` | Relational DB | — (external, from config) | H2 by default; MySQL or Postgres with a profile | `src/main/resources/application.properties:2-4`, `application-mysql.properties:2-3`, `application-postgres.properties:2-3`, `pom.xml:76-80` | VERIFIED |
| `cache` | vets cache | — (external, from config) | Spring cache named `vets` | `vet/VetRepository.java:45,55`; the Caffeine provider comes from `pom.xml:67-70,81-85` and AGENTS.md (checked by the author through `/actuator/caches`) | `@Cacheable` VERIFIED; provider "Caffeine, unbounded" INFERRED (from AGENTS.md; not run in this session) |

## Edges

| Edge id | From → To | Label | Source | Status |
|---|---|---|---|---|
| `http-owners` | browser → owner-web | `/owners/**` | `owner/OwnerController.java:72-178` (mappings), `owner/PetController.java:48,100-144` | VERIFIED |
| `http-visits` | browser → visit-web | `…/visits/new` | `owner/VisitController.java:90,97` | VERIFIED |
| `http-vets` | browser → vet | `/vets.html` | `vet/VetController.java:45,72` | VERIFIED |
| `http-system` | browser → system | `/, /oups, ?lang` | `system/WelcomeController.java:25`, `system/CrashController.java:31`, `system/WebConfiguration.java:44-58` (the `lang` interceptor runs on every request) | VERIFIED |
| `owner-load-save` | owner-web → owner-repo | `findById / save` | cgr `callers OwnerRepository.findById(Integer)` → `OwnerController.java:67,181`, `PetController.java:69,83`; cgr `callers findByLastNameStartingWith` → `OwnerController.java:145`; grep: `OwnerController.java:84,168` `save`, `PetController.java:126,199` `saveAndFlush` | VERIFIED |
| `visit-load-save` | visit-web → owner-repo | `findById / save` | cgr `callers OwnerRepository.findById(Integer)` → `VisitController.java:66`; `VisitController.java:109` `save` | VERIFIED |
| `pet-types` | owner-web → pettype | `findPetTypes` | cgr `callers PetTypeRepository.findPetTypes()` → `PetController.java:64` (`@ModelAttribute("types")`, line 62) and `PetTypeFormatter.java:53` | VERIFIED. That Spring MVC uses `PetTypeFormatter` to bind the pet form's `type` field is INFERRED (framework wiring through `@Component`; there is no explicit registration) |
| `repo-aggregate` | owner-repo → owner-agg | `persists Owner` | `owner/OwnerRepository.java:36` (`JpaRepository<Owner, Integer>`); also cgr `callers Owner.addPet` → `PetController.java:103,125,197`, `callers Owner.addVisit` → `VisitController.java:108`, `callers Owner.getPet(Integer)` → `PetController.java:86,189`, `VisitController.java:70` | VERIFIED |
| `repo-sql` | owner-repo → db | `JPA, cascade ALL` | `owner/Owner.java:67` and `owner/Pet.java:56` (`@OneToMany(cascade = ALL)`); `application.properties:2-4,10` | Cascades VERIFIED. The SQL is generated by Spring Data JPA and Hibernate, so it is not in the code graph: INFERRED from the framework |
| `vet-cache` | vet → cache | `@Cacheable` | `vet/VetRepository.java:45,55`; cgr `callers VetRepository.findAll()` → `VetController.java:77`, `findAll(Pageable)` → `VetController.java:69` | VERIFIED |
| `vet-sql` | vet → db | `JPA read` | `vet/VetRepository.java:38,44,54` (`Repository<Vet,Integer>`, `@Transactional(readOnly = true)`) | INFERRED (Spring Data generates the query; drawn dashed) |
| `agg-extends` | owner-agg → base | `extends` | cgr `importers model.Person` → `owner/Owner.java:23`; `importers model.NamedEntity` → `owner/Pet.java:24`, `owner/PetType.java:18`; `importers model.BaseEntity` → `owner/Visit.java:21` | VERIFIED |

## Deliberately not drawn

| Item | Why | Source | Status |
|---|---|---|---|
| vet → base (`Vet extends Person`, `Specialty extends NamedEntity`) | Left out to keep the routes readable; it is listed on the diagram's "Not drawn" card | cgr `importers model.Person` → `vet/Vet.java:25`; `importers model.NamedEntity` → `vet/Specialty.java:18`, `vet/Vet.java:24` | VERIFIED |
| pettype → db | Same JPA path as `repo-sql`; left out to reduce edges | `owner/PetTypeRepository.java:30` | INFERRED |
| Application Bootstrap (CodeBoarding `5`) | It wires the app but has no runtime dependency to draw. `PetClinicRuntimeHints` registers `BaseEntity`, `Person` and `Vet` for native images | `PetClinicApplication.java:28-29`, `PetClinicRuntimeHints.java:33-35` | VERIFIED |
| CacheConfiguration → vets cache | A `JCacheManagerCustomizer` that never applies, because no JCache provider is on the classpath and Boot picks Caffeine | `system/CacheConfiguration.java:31-37`; `pom.xml:67-70,81-85`; AGENTS.md | INFERRED (from AGENTS.md and the pom; not checked at runtime in this session) |

## Where CodeBoarding disagrees with the code

1. **`1.1` "Owner & Pet Web Management"** puts controllers (`OwnerController`, `PetController`) together with the `Owner` entity, `PetType` and `OwnerRepository`, so it mixes web, domain and persistence. The map splits it into `owner-web`, `owner-repo` and `owner-agg`.
2. **`1.4` "Pet Domain Entity"** as its own component, and the relation "Visit Scheduling → Pet Domain Entity: records visit on pet", hide the aggregate. `VisitController` does call `Pet.addVisit` (`VisitController.java:79`, a new unsaved Visit bound to the form), but it persists only through `Owner.addVisit` → `OwnerRepository.save` (`VisitController.java:108-109`). There is no Pet or Visit repository, so Pet and Visit are one aggregate under Owner.
3. **The relation "Visit Scheduling → Owner & Pet Web Management: loads owner for visit context"** points at the wrong target. `VisitController` never calls `OwnerController` or `PetController`. It calls `OwnerRepository` directly (cgr callers, `VisitController.java:66`).
4. **`3` "System Infrastructure"** lists `CacheConfiguration` as if it configured the vets cache. According to AGENTS.md it has no effect at runtime (see above).
5. CodeBoarding records **no external systems** (database, cache) and no edges for `2` or `3` apart from "extends base entity classes". Those edges were added from the code.
