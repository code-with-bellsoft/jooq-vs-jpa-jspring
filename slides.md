---
theme: seriph
title: "The Persistence Heavyweight Championship: jOOQ vs JPA"
background: /boxers.png
class: text-center
lineNumbers: false
drawings:
  persist: false
transition: slide-left
colorSchema: dark
download: true
aspectRatio: 16/9
canvasWidth: 980
highlighter: shiki
addons:
  - "@slidev-polls/component"
pollServer: https://polls.asm0dey.site
---

# The Persistence Heavyweight<br/>Championship:<br/>jOOQ vs JPA

<div class="mt-12 grid grid-cols-2 gap-8 text-lg opacity-90">
  <div>Pasha Finkelshteyn</div>
  <div>Catherine Edelveis</div>
</div>


---
layout: two-cols
class: text-2xl
---

## <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> corner

<v-click>

SQL is the **only thing** you need

</v-click>

::right::

## <span class="text-amber-400">JPA</span> corner

<v-click>

Usually, you **don't need to know SQL**

</v-click>


---
layout: center
class: text-center
---

# Imagine you are starting a new project!

<div class="mt-8 text-3xl font-bold tracking-wider">
  WE ARE FIGHTING FOR YOUR FUTURE
</div>


---
layout: section
class: text-center
---

# ROUND 1

## Speed (and Ease) of Development

---

# JPA — write domain models

```java {1-3|5-7|9-11}
@Entity
@Table(name = "suppliers")
public class Supplier {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @OneToMany(mappedBy = "supplier", cascade = CascadeType.ALL,
               orphanRemoval = false)
    private List<Product> products = new ArrayList<>();
}
```


- DB assigns primary key on insert
- Define connections with annotations



---

# JPA — repository + service

```java {1-2|4-9}
public interface ProductRepository extends JpaRepository<Product, Long> {
}

@Service
@Transactional
public class ProductService {
    public Optional<Product> findById(Long productId) {
        return productRepository.findById(productId);
    }
}
```


Schema generation, SQL generation and execution, DB connection, fetching and mapping — **DONE FOR YOU**


---

# JPA — derived queries

```java {3}
public interface ProductRepository extends JpaRepository<Product, Long> {

    List<Product> findBySupplierIdOrderByNameDesc(Long supplierId);
}
```


Queries derived from method names


---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ — active record

```java {1-2|3|4}
AuthorRecord author =
    db.fetchOne(AUTHOR, AUTHOR.ID.eq(1));
// do your thing
author.store();
```


---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ — typed DSL query

```java {all|3|4|5|6}
public List<ProductRecord> findBySupplier(Long supplierId) {
  return dsl
    .selectFrom(PRODUCTS)
    .where(PRODUCTS.SUPPLIER_ID.eq(supplierId))
    .orderBy(PRODUCTS.NAME.desc())
    .fetchInto(ProductRecord.class);
}
```


---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="f1d14932-fe9d-469c-ba5c-7098415ba4be"
/>

---
layout: section
class: text-center
---

# ROUND 2

## Type-Safety

---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ — generate everything from DB

- `PRODUCT` — holds `PRODUCT.NAME`, `PRODUCT.ID` etc.
- `ProductRecord` — fetch/insert/update into DB
- `ProductDao` — CRUD without SQL <span class="opacity-60">(optional)</span>
- `Product` — POJO <span class="opacity-60">(optional)</span>
- `IProduct` — interface implemented by them all <span class="opacity-60">(optional)</span>

---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ codegen plugin

```xml {2-4|7-11}
<plugin>
    <groupId>org.jooq</groupId>
    <artifactId>jooq-codegen-maven</artifactId>
    <version>3.21.1</version>
    <executions>
        <execution>
            <id>generate-jooq</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>generate</goal>
            </goals>
        </execution>
    </executions>
```

---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ codegen — DB config

```xml {1-6|8-13}
<jdbc>
    <driver>org.postgresql.Driver</driver>
    <url>jdbc:postgresql://localhost:5432/yourdb</url>
    <user>youruser</user>
    <password>yourpassword</password>
</jdbc>
<generator>
    <database>
        <name>org.jooq.meta.postgres.PostgresDatabase</name>
        <includes>.*</includes>
        <excludes/>
        <inputSchema>public</inputSchema>
    </database>
```

---

# JPA — Hibernate Processor

- Generates JPA **static metamodel**
- Validates HQL, JPQL, JDQL queries
- Type-safe everything: Criteria queries, entity graphs, dynamic sorting/filtering

---

# JPA — annotation processor setup

```xml {6-12}
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <version>3.13.0</version>
    <configuration>
        <annotationProcessorPaths>
            <path>
                <groupId>org.hibernate.orm</groupId>
                <artifactId>hibernate-processor</artifactId>
                <version>7.3.0.Final</version>
            </path>
        </annotationProcessorPaths>
        <annotationProcessorPathsUseDepMgmt>true</annotationProcessorPathsUseDepMgmt>
    </configuration>
</plugin>
```

---

# JPA — generated static metamodel

```java
@StaticMetamodel(Product.class)
@Generated("org.hibernate.processor.HibernateProcessor")
public abstract class Product_ {

    public static final String ID = "id";

    public static volatile EntityType<Product> class_;

    public static volatile SingularAttribute<Product, Long> id;
}
```

---

# JPA — Criteria with metamodel

```java {6-7}
public List<Product> findByName(String name) {
    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Product> cq = cb.createQuery(Product.class);

    Root<Product> product = cq.from(Product.class);
    cq.select(product)
            .where(cb.equal(product.get(Product_.name), name));

    return entityManager.createQuery(cq).getResultList();
}
```


---

# JPA — Dynamic Queries with Criteria API

```java
public List<Product> searchProducts(String supplierName,
                                    String categoryName,
                                    int offset,
                                    int limit) {

    CriteriaBuilder cb = entityManager.getCriteriaBuilder();
    CriteriaQuery<Product> cq = cb.createQuery(Product.class);
    Root<Product> product = cq.from(Product.class);
```

---

# JPA — Build predicates

```java {1|3-7}
Predicate predicate = cb.conjunction();

if (supplierName != null && !supplierName.isBlank()) {
    Join<Product, Supplier> supplier = product.join(Product_.supplier, JoinType.LEFT);
    predicate = cb.and(predicate,
                       cb.equal(supplier.get(Supplier_.name), supplierName));
}

if (categoryName != null && !categoryName.isBlank()) {
    Join<Product, Category> category = product.join(Product_.category, JoinType.LEFT);
    predicate = cb.and(predicate,
                       cb.equal(category.get(Category_.name), categoryName));
}
```

---

# JPA — Apply predicate

```java {3}
cq.select(product)
        .distinct(true)
        .where(predicate)
        .orderBy(cb.asc(product.get(Product_.name)));

return entityManager.createQuery(cq)
        .setFirstResult(offset)
        .setMaxResults(limit)
        .getResultList();
}
```

---

# JPA — Query Validation

```java {all}
@NamedQuery(
    name = "brokenOrderQuery",
    query = "select o from CustomerOrder o where o.noSuchField = :id"
)
```

```text {all|5-6}
[INFO] -------------------------------------------------
[INFO] BUILD FAILURE
[INFO] -------------------------------------------------
[ERROR] Failed to execute goal maven-compiler-plugin:3.13.0:compile
[ERROR] CompileTimeQueryValidationDemo.java:[7,17]
        Could not resolve attribute 'noSuchField' of 'CustomerOrder'
```



---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> jOOQ — typed join query

```java {all|2|3|4|5|6-8}
db
  .select(PRODUCT.NAME)
  .from(PRODUCT)
  .join(SUPPLIER).on(PRODUCT.SUPPLIER_ID.eq(SUPPLIER.ID))
  .where(SUPPLIER.NAME.eq(supplierName))
  .orderBy(PRODUCT.NAME.desc())
  .limit(limit)
  .offset(offset)
  .fetchInto(Product.class)
```

---

# <span class="font-mono font-bold text-white bg-black px-2 py-0.5 rounded">jOOQ</span> Implicit JOIN

```java {2|3}
dsl
  .selectFrom(PRODUCT)
  .where(PRODUCT.supplier().NAME.equalIgnoreCase(supplierName))
  .orderBy(PRODUCT.NAME.desc())
  .limit(limit)
  .offset(offset)
  .fetchInto(Product.class)
```


---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="251f3114-c36f-4052-892e-9a2e1ec0af60"
/>

---
layout: section
class: text-center
---

# ROUND 3

## N+1 Management

---
layout: center
class: text-center
---

# N+1

<div class="text-2xl mt-4 max-w-2xl">

SELECT that performs a **separate SQL select** to load the data is generally termed N+1

</div>

---

# N+1 = Fetch Strategy Problem

```java
for (Product product : products) {
    List<Supplier> suppliers = product.getSuppliers();
    // N more queries fire here
}
```

---

# JPA — guardrails

- `FetchType.LAZY` explicitly for all associations
- `JOIN FETCH` in the query
- `@BatchSize` annotation for batched fetching

---

# JPA — Fetch with Criteria

```java {all|3}
CriteriaQuery<CustomerOrder> cq = cb.createQuery(CustomerOrder.class);
Root<CustomerOrder> order = cq.from(CustomerOrder.class);
Fetch<CustomerOrder, OrderItem> items = order.fetch(CustomerOrder_.items);
cq.select(order)
  .distinct(true)
  .where(cb.equal(order.get(CustomerOrder_.id), orderId));
```

<img v-click src="./lamba.png" class="w-1/2 mx-auto" />

---
layout: center
class: text-center
---

# jOOQ verdict


<div class="relative inline-block">
  <img src="./problems.png" class="max-h-[70vh]" />

<img
v-click
src="./non1.jpg"
class="absolute top-0 left-0 z-10 w-full h-full object-contain"
/>
</div>


---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="6924a627-39f0-4a98-9be4-7262d41ec35a"
/>

---
layout: section
class: text-center
---

# ROUND 4

## Write-Side Query Complexity

---
layout: quote
class: text-center
---

> If your writing becomes complex — load a complex object graph with 20 entities,
> perform optimistic locking, modify it in many ways and persist it in one go —
> then SQL / jOOQ will not help you. **This is what Hibernate has originally been created for.**

— <em>Lukas Eder, creator of jOOQ</em>

---

# JPA — Request payload

```java
public record OrderEditRequest(
    ShipmentAddressRequest address,
    Map<Long, Integer> newProductIdsWithQuantities,
    List<String> productUpisToRemove,
    Map<String, Integer> quantityUpdates,
    String couponCode,
    BigDecimal couponDiscountAmount
) { }
```

---

# JPA — revise order

```java {all|3|7-8}
@Transactional
public void reviseOrder(Long orderId, OrderEditRequest request) {
    CustomerOrder order = repository.fetchOrderGraphForEdit(orderId);

    ShipmentAddressRequest address = request.address();
    Country newCountry = repository.findCountryByCode(address.countryCode());
    // Update shipping address
    order.changeShipping(address.city(), address.street(),
                         address.postalCode(), newCountry);
```



---

# JPA — remove items

```java {all|7}
// Remove existing items -> orphanRemoval deletes rows
List<String> productUpisToRemove = request.productUpisToRemove() == null
        ? List.of()
        : request.productUpisToRemove();

for (String productUpi : productUpisToRemove) {
    order.removeItemByProductUpi(productUpi);
}
```

---

# JPA — add items

```java {all|9-10}
// Add new items -> cascade persist inserts rows
Map<Long, Integer> newProducts = request.newProductIdsWithQuantities() == null
        ? Map.of()
        : request.newProductIdsWithQuantities();

for (Map.Entry<Long, Integer> entry : newProducts.entrySet()) {
    Long productId = entry.getKey();
    Integer quantity = entry.getValue();
    order.addItem(entityManager.getReference(Product.class, productId),
                  quantity);
}
```

---

# JPA — change quantity

```java {all|9-13}
// Change quantity of existing items
Map<String, Integer> quantityUpdates = request.quantityUpdates() == null
        ? Map.of()
        : request.quantityUpdates();

for (Map.Entry<String, Integer> entry : quantityUpdates.entrySet()) {
    String productUpi = entry.getKey();
    Integer quantity = entry.getValue();
    if (quantity == 0) {
        order.removeItemByProductUpi(productUpi);
    } else {
        order.updateItemQuantity(productUpi, quantity);
    }
}
```

---

# JPA — coupon + recalc

```java {all|3-5|7,8}
// Add coupon -> cascade persist
if (request.couponCode() != null) {
    order.addCoupon(request.couponCode(), request.couponDiscountAmount());
}

// Recalculate derived business value
order.recalculateTotal();
// That's it! No need to explicitly save updated objects
}
```

<v-click>

If something changes in the DB → **Optimistic locking via `@Version`**

</v-click>

---

# jOOQ — update address

```java {all|3-9|10-12}
public void reviseOrder(Long orderId, OrderEditRequest request) {
    // Update shipping address in-place — no order graph needed
    ShipmentAddressRequest address = request.address();
    int updated = dsl.update(CUSTOMER_ORDER)
            .set(CUSTOMER_ORDER.CITY, address.city())
            .set(CUSTOMER_ORDER.STREET, address.street())
            .set(CUSTOMER_ORDER.POSTAL_CODE, address.postalCode())
            .set(CUSTOMER_ORDER.COUNTRY_ID, newCountryId)
            .where(CUSTOMER_ORDER.ID.eq(orderId))
            .execute();
    if (updated == 0) {
        throw new IllegalArgumentException("Order not found: " + orderId);
    }
```

---

# jOOQ — remove items by UPI

```java
// 3. Remove items by UPI
List<String> upisToRemove = request.productUpisToRemove() == null
        ? List.of()
        : request.productUpisToRemove();

if (!upisToRemove.isEmpty()) {
    dsl.delete(ORDER_ITEM)
            .where(ORDER_ITEM.ORDER_ID.eq(orderId))
            .and(ORDER_ITEM.PRODUCT_UPI.in(upisToRemove))
            .execute();
}
```

---

# jOOQ — insert new items

```java
// 4. Insert new items
Map<Long, Integer> newProducts = request.newProductIdsWithQuantities() == null
    ? Map.of()
    : request.newProductIdsWithQuantities();

if (!newProducts.isEmpty()) {
    var insertStep = dsl.insertInto(ORDER_ITEM,
        ORDER_ITEM.ORDER_ID,
        ORDER_ITEM.PRODUCT_ID,
        ORDER_ITEM.QUANTITY);

    newProducts.entrySet().stream()
        .forEach(e -> insertStep.values(orderId, e.getKey(), e.getValue()));

    insertStep.execute();
}
```

---

# jOOQ — update quantities

```java
if (!upisToUpdate.isEmpty()) {
    var quantityCase = DSL.case_(ORDER_ITEM.PRODUCT_UPI);
    for (var entry : upisToUpdate) {
        quantityCase = quantityCase.when(entry.getKey(), DSL.val(entry.getValue()));
    }

    dsl.update(ORDER_ITEM)
            .set(ORDER_ITEM.QUANTITY, quantityCase.end())
            .where(ORDER_ITEM.ORDER_ID.eq(orderId))
            .and(ORDER_ITEM.PRODUCT_UPI.in(
                    upisToUpdate.stream().map(Map.Entry::getKey).toList()))
            .execute();
}
```

---

# jOOQ — delete + update loops

```java
if (!upisToDelete.isEmpty()) {
    dsl.delete(ORDER_ITEM)
            .where(ORDER_ITEM.ORDER_ID.eq(orderId))
            .and(ORDER_ITEM.PRODUCT_UPI.in(upisToDelete))
            .execute();
}

for (var entry : upisToUpdate) {
    dsl.update(ORDER_ITEM)
            .set(ORDER_ITEM.QUANTITY, entry.getValue())
            .where(ORDER_ITEM.ORDER_ID.eq(orderId))
            .and(ORDER_ITEM.PRODUCT_UPI.eq(entry.getKey()))
            .execute();
}
```

---

# jOOQ — replace coupon

```java
// 6. Replace coupon — delete existing, insert new if present
dsl.delete(ORDER_COUPON)
        .where(ORDER_COUPON.ORDER_ID.eq(orderId))
        .execute();

if (request.couponCode() != null) {
    dsl.insertInto(ORDER_COUPON,
                   ORDER_COUPON.ORDER_ID,
                   ORDER_COUPON.CODE,
                   ORDER_COUPON.DISCOUNT_AMOUNT)
            .values(orderId, request.couponCode(),
                    request.couponDiscountAmount())
            .execute();
}
```

---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="ea1ae343-b579-43f0-8642-442dee838ab2"
/>

---
layout: section
class: text-center
---

# ROUND 5

## Runtime Performance

---

# JPA — Performance features

- Caching
- Stateless session
- Projection fetching

<v-click>

**Complex tool. Complex ≠ fragile.**

</v-click>

<img v-click="2" src="./accident.png" class="w-1/2 mx-auto" />


---

# jOOQ — query cost

- Takes some time to construct jOOQ queries
- Takes some time to render SQL strings
- Takes some time to bind values to PreparedStatement

<div class="mt-8 text-xl opacity-90">
  → Use connection pooling
</div>

<img v-click src="./efishenci.png" class="w-1/2 mx-auto" />


---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="8c2d2353-3248-4ea1-9420-356cb96857df"
/>

---
layout: section
class: text-center
---

# ROUND 6

## Schema Evolution and Refactoring Resilience

---

# JPA — Domain-first codegen + compile-time checks

- Static metamodel is **refactor-safe** against domain changes
  - Rename `supplier → vendor`, regenerate metamodel — compiler detects broken queries
- `jakarta.persistence.schema-generation.database.action=validate` + integration tests to avoid compiling with schema mismatches
- DB migration tooling, schema diff checks for DB/entities mismatch

---

## jOOQ

Change your DB incompatibly and see how your application doesn’t even compile. <br/>
Because it **shouldn’t**.<br/>
And you don’t need plugins for it!

<img v-click src="./winnie.png" class="w-1/2 mx-auto" />

---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="d06fcb01-85c2-487d-af4b-6da92d693c1e"
/>

---
layout: section
class: text-center
---

# ROUND 7

## Pricing and Maintenance

---

# jOOQ Editions

| Edition | DBs | Price |
|---|---|---|
| **Open Source** | Open source DBs (PostgreSQL, MySQL, H2…) | free |
| **Express** | + versioned RDBMS / Java support | 99 € |
| **Professional** ⭐ | + Oracle, SQL Server, MS Access | 399 € |
| **Enterprise** | enterprise DBs | 799 € |

<div class="mt-4 text-sm opacity-80">per floating developer workstation/year, incl. maintenance & support</div>

---

# JPA — Project Maturity

- Funded/sponsored for **~20 years**
- Corporate contributors: Red Hat, Oracle, MongoDB, etc.
- Multiple books
- Stack Overflow:
  - hibernate: **95K** questions
  - jpa: **52K** questions
  - spring-data-jpa: **23K** questions

<div class="mt-2 opacity-80">Used by 466k projects · 627 contributors</div>

---

# JPA — Licensing and Support

- **Apache 2.0** license → use in commercial products for free with ANY database supported by Hibernate
- Support with SLA and EOL available from **Red Hat**

---
layout: center
class: text-center
---

<PollResults
  slug="jpa-jooq-jspring"
  pollId="d53d36a7-9c53-4363-9473-a6c8f44bcf97"
  questionId="e90ef410-d08b-437f-934d-7bb713d31a13"
/>


---
layout: center
class: text-center
---

# Thank You!

<div class="mt-8 text-2xl">Watch us — CyberJAR</div>

<div class="mt-8 grid grid-cols-2 gap-8 text-lg opacity-90">
</div>


<img src="./cyberjar.png" class="w-1/2 mx-auto" />

---

# What we didn't mention

1. Read-side queries
2. Multiset
3. Custom SQL operations
4. Query Validator
5. Blaze Persistence