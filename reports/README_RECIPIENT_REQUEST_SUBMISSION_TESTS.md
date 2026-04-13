# Recipient Request Submission Tests

This directory contains automated tests for the Recipient Request Submission feature, ensuring that critical business rules like weekly allowance, provider capacity, and ownership are enforced.

## 🧪 Test Coverage

The tests are located in `tests/Feature/Recipient/RecipientRequestSubmissionTest.php` and cover the following scenarios:

1.  **Happy Path**: Verifies that a recipient can successfully submit a valid request.
    -   Checks redirection and database insertion.
    -   Verifies correct status (`REQUESTED`) and price snapshot.

2.  **Allowance Exceeded**: Verifies the strict 400 SAR weekly allowance.
    -   Sets up existing requests totaling 380 SAR.
    -   Attempts to add 50 SAR -> Blocks creation.
    -   Asserts exact error message: "You have exceeded your weekly allowance of 400 SAR."
    -   Confirms `REQUESTED` status items do NOT count towards the limit.

3.  **Date Boundary**: Verifies that requests from previous weeks do not count towards the current week's allowance.

4.  **Wrong Ownership**: prevents requesting a menu item that belongs to a different provider.

5.  **Inactive Item**: Prevents requesting an item with `is_active = false`.

6.  **Capacity Off**: Prevents requesting from a provider with `daily_capacity <= 0`.

7.  **Authorization**: Ensures non-recipients (e.g., donors, guests) cannot submit requests.

## How to Run

Run the tests using Laravel's Artisan test runner:

```bash
# Run all tests in the file
php artisan test --filter=RecipientRequestSubmissionTest
```

## Prerequisites & Notes

-   **Database**: Uses `RefreshDatabase` trait, so it requires a configured database connection (usually in `phpunit.xml` or `.env.testing`). SQLite in-memory is recommended for speed if supported by your project configuration.
-   **Time Handling**: Tests use `Carbon::setTestNow()` to freeze time, ensuring determinstic results for weekly allowance calculations (testing on Wednesday to catch week boundaries safely).
-   **Factories**: Tests rely on `User` factory and create `ProviderOperatingInfo` / `ProviderMenuItem` manually as no factories currently exist for them.
