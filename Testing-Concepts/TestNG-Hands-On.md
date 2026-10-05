TestNG — Explained with Rentora

**1. Clean Definition**

TestNG (Test Next Generation) is a Java testing framework — similar to JUnit but with more powerful features — used to write, organize, and execute automated tests. It's commonly paired with Selenium for automation testing and is widely used in real companies for structuring large test suites.

**2. Step-by-Step Breakdown of Each Feature**
Step 1: Annotations

Annotations are special markers (@Test, @BeforeMethod, etc.) placed above methods to tell TestNG when and how to run them — without annotations, TestNG wouldn't know which methods are tests.

*Annotation*	                     **When It Runs**
@BeforeSuite	          Once, before the entire test suite
@BeforeClass	          Once, before the first test in a class
@BeforeMethod	          Before every test method
@Test	                  The actual test method
@AfterMethod	          After every test method
@AfterClass	            Once, after all tests in a class finish
@AfterSuite	            Once, after the entire suite finishes

Step 2:                 Test Execution

TestNG runs tests based on annotation order (not just top-to-bottom in the file), and lets you control execution via an XML config file (testng.xml) instead of hardcoding which tests run.

Step 3: Grouping

You can tag tests into groups (e.g., "smoke", "regression", "sanity") and run only a specific group instead of the entire suite — directly connects to your earlier Smoke vs Regression topic.

Step 4: Dependencies

You can make one test depend on another passing first (e.g., "don't run Booking test unless Login test passed") using dependsOnMethods.

Step 5: Data-Driven Testing

Using @DataProvider, you can run the same test multiple times with different sets of input data — directly connects to your Boundary Value Analysis / Equivalence Partitioning topics, since you can feed multiple boundary values into one test automatically.

Step 6: Parallel Execution

TestNG can run multiple tests simultaneously (e.g., on Chrome and Firefox at the same time) instead of one after another — saves huge amounts of time in large regression suites.

Step 7: Reports

TestNG automatically generates HTML/XML reports after execution, showing Pass/Fail/Skipped counts — useful for sharing results with the team without manually tracking every test.

**3. Simple Example**

java
public class LoginTest {
    @BeforeMethod
    public void setup() {
        System.out.println("Opening browser..."); // runs before every test
    }
    @Test(groups = {"smoke"})
    public void testValidLogin() {
        System.out.println("Testing valid login");
    }
    @Test(groups = {"regression"}, dependsOnMethods = {"testValidLogin"})
    public void testBookingAfterLogin() {
        System.out.println("Testing booking — only runs if login test passed");
    }
    @AfterMethod
    public void teardown() {
        System.out.println("Closing browser..."); // runs after every test
    }
}

Explanation: @BeforeMethod/@AfterMethod wrap every test (open/close browser each time). groups = {"smoke"} lets you run just smoke tests later. dependsOnMethods ensures testBookingAfterLogin only runs if login succeeded first — no point testing booking if login itself is broken.

**4. Real-Time Project Scenario (Rentora)**
Annotations + Execution Applied to Rentora
java

    public class RentoraBookingTest {
        WebDriver driver;

    @BeforeMethod
    public void setup() {
        driver = new ChromeDriver();
        driver.get("http://localhost:8080/login");
    }

    @Test(groups = {"smoke"}, priority = 1)
    public void testLogin() {
        driver.findElement(By.id("email")).sendKeys("dharun@mail.com");
        driver.findElement(By.id("password")).sendKeys("test123");
        driver.findElement(By.id("loginBtn")).click();
        Assert.assertEquals(driver.getTitle(), "Dashboard");
    }

    @Test(groups = {"regression"}, dependsOnMethods = {"testLogin"}, priority = 2)
    public void testBooking() {
        driver.findElement(By.id("carSelect")).sendKeys("Car A");
        driver.findElement(By.id("confirmBtn")).click();
        Assert.assertTrue(driver.findElement(By.id("successMsg")).isDisplayed());
    }

    @AfterMethod
    public void teardown() {
        driver.quit();
       }
    }


**Data-Driven Testing Applied to Rentora's Date-Overlap Bug**

Instead of writing separate test methods for each boundary value (0, 1, 29, 30, 31 days — from your BVA topic), you feed them all through one test using @DataProvider:

java
@DataProvider(name = "bookingDurations")
public Object[][] getDurations() {
    return new Object[][] {
        {0, false},   // invalid
        {1, true},    // min boundary - valid
        {30, true},   // max boundary - valid
        {31, false}   // invalid
    };
}

@Test(dataProvider = "bookingDurations")
public void testBookingDuration(int days, boolean expectedResult) {
    boolean actualResult = bookingService.isValidDuration(days);
    Assert.assertEquals(actualResult, expectedResult);
}

→ This runs the same test 4 times, once per row — this is exactly how your BVA theory (from earlier) becomes real automated code, instead of manually writing 4 separate test methods.

Grouping Applied to Rentora
xml
<!-- testng.xml -->
<suite name="RentoraSuite">
    <test name="SmokeTests">
        <groups>
            <run>
                <include name="smoke"/>
            </run>
        </groups>
        <classes>
            <class name="RentoraBookingTest"/>
        </classes>
    </test>
</suite>

→ Before a new build is deployed, you'd run only the "smoke" group first (connects directly to your Smoke Testing topic) — if it passes, then you run the full "regression" group.

Parallel Execution Applied to Rentora
xml
<suite name="RentoraSuite" parallel="tests" thread-count="2">
    <test name="ChromeTest">
        <parameter name="browser" value="chrome"/>
        <classes><class name="RentoraBookingTest"/></classes>
    </test>
    <test name="FirefoxTest">
        <parameter name="browser" value="firefox"/>
        <classes><class name="RentoraBookingTest"/></classes>
    </test>
</suite>

→ Runs the same booking tests on Chrome and Firefox simultaneously instead of one after another — cuts regression time in half.

**Reports**

After running the above, TestNG auto-generates test-output/index.html showing:

Total: 10 | Passed: 8 | Failed: 1 | Skipped: 1

→ You'd use this to report results to your project guide or team instead of manually listing which tests passed.


**5. How TestNG Connects to Everything You've Already Learned**

TestNG Feature	Connects To
Grouping (smoke/regression)	Smoke vs Regression Testing topic
Data-Driven Testing	Boundary Value Analysis / Equivalence Partitioning
Dependencies	Test execution order, Retesting logic
Reports	Pass/Fail metrics, test documentation

This is a strong interview point: "I didn't just learn testing techniques in theory — I automated them using TestNG's @DataProvider to run BVA test cases for Rentora's booking duration, and used groups to separate smoke tests from full regression."
