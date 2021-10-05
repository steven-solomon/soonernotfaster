---
title: "Thinking Through Test Pollution"
date: 2021-10-05T19:20:41-04:00
---

When you run tests in a random class and method order, which results in them failing, you have demonstrated test pollution. This is usually the result of sloppy tests that don't clean up after themselves. Taking the time to understand the failures can help point you towards the root cause of pollution. Recipients of pollution are either `Beneficiaries` or `Innocent Bystanders`.

## Beneficiaries

`Beneficiaries` are those tests that originally took advantage of test pollution, baking it into the assumptions used to write the test. Now that the tests are running in a different order, the source of pollution is executed at a later time, leaving the `Beneficiary` as a failing test. This type of pollution recipient fails when run in isolation.

### How to fix it

For this type of pollution you are looking for tests that setup the behavior missing from the currently failing test. In the list of randomly executed classes and methods, the culprit is likely after the failing test. Once you find the missing setup, you will have to decide how to isolate the pollution source and how to move that setup to the test that desires it. Sometimes this requires duplicating the setup since it may be needed in the location that uses it.

## Innocent Bystanders

Innocent Bystanders are a category of tests which demonstrate pollution by having their assumptions about the environment fail. For instance, a test could assume that a repository will have no instances of an entity, but due to test pollution three are returned. You can identify `Innocent Bystanders` because they are tests that pass when run in isolation, but fail when run in the seeded order you have discovered.

### How to fix it

Locating the offending test for this type requires you to look before the current test. Looking for any test that uses the same resource which is at the heart of the failure. One example would be a specific database table has the incorrect number of rows. In order to find the pollution source, you will look for code in the system that also modifies that table, and next see which tests utilize those elements.

In order to identify the source of pollution, identify the shared dependencies. Then find which tests also depend on those resources and have been executed before the failure in the randomized order.

# Making the world green again

Fixing tests can be tricky but with both types of symptoms one solution you always need is to isolate the pollution source to prevent more issues. Thankfully, Spring has some handy tools to make this possible. The first way to clean up any database changes following the test, this is `@Transactional`. Although, `@Transactional` is usually used inside of classes that interact with the database, but it can allow the next test method to have a clean start. The transactional behavior is baked into the second option, `@Data<datasource>Test`. These include `@DataJpaTest`, and `@DataJdbcTest`, and are used in concert with `@SpringBootTest`.