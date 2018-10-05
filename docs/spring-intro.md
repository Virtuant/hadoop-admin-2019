## Introduction to Core Spring

**Objective:** In this lab you'll come to understand the basic workings of the _Reward Network_ reference
application and you'll be introduced to the tools you'll use throughout the course. Once you will have familarized yourself with the tools and the application domain, you will implement and test the [rewards application](https://virtuant.github.io/spring-core/reference-domain.html) using Plain Old Java objects (POJOs).

**Successful outcome:** At the end of the lab you will see that the application logic will be clean and not coupled with infrastructure APIs. You'll understand that you can develop and unit test your business logic without using Spring. Furthermore, what you develop in this lab will be directly runnable in a Spring environment without change.

**File location:** `~/labs`

>**Note:** In every lab, read to the end of eachnumbered section before doing anything. There are often tips and notes to help you, but they may be just over the next page or off the bottom of the screen. If you get stuck, don't hesitate to ask for help!

##### What you will learn:

1. Basic features of the Spring Tool Suite
2. Core _RewardNetwork_ Domain and API
3. Basic interaction of the key components within the domain

---
### Steps



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>1. Getting Started with the Spring Tool Suite</h2>

The Spring Tool Suite (STS) is a free IDE built on the Eclipse Platform. In this section, you will become familiar with the Tool Suite. You will also understand how the lab projects have been structured.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>2. Launch the Tool Suite</h2>

Launch the Spring Tool Suite by using the shortcut link on your desktop.

![sts-thumb](https://user-images.githubusercontent.com/558905/40527986-16a19c7c-5fbd-11e8-8672-8c1318a613db.png)

After double-clicking the shortcut, you will see the STS splash image appear.

![spring-splash](https://user-images.githubusercontent.com/558905/40527985-1694832a-5fbd-11e8-8d9c-20534f534e52.png)


You will be asked to select a workspace. You should accept the default location offered. You can optionally check the box labeled _use this as the default and do not ask again._


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>3. Understanding the Eclipse/STS project structure</h2>

If you've just opened STS, it may be still starting up. Wait several moments until the progress indicator on the bottom right finishes. When complete, you should have no red error markers within the _Package Explorer_or _Problems_ views

Now that STS is up and running, you'll notice that, within the _Package Explorer_ view on the
left, projects are organized by _WorkingSets_. Working Sets are essentially folders that contain a group of Eclipse projects. These working sets represent the various labs you will work through during this course. Notice that they all begin with a number so that the labs are organized in order as they occur in this lab guide.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>4. Browse Working Sets and projects</h2>

If it is not already open, expand the _01-spring-intro_ Working Set.
Within you'll find two projects: _spring-intro_ and _spring-intro-solution_. _spring-intro_ corresponds to the
start project. This pair of _start_ and _solution_ projects is a common
pattern throughout the labs in this course.

Open the _spring-intro_ project and expand its _ReferencedLibraries_ node. Here you'll see a number of dependencies similar to the screenshot below:

![dependencies](https://user-images.githubusercontent.com/558905/40527973-1607bec2-5fbd-11e8-9279-e0ae054edbf1.png)

>**Tip:** This screenshot uses the "Hierarchical" Package Presentation view instead of the "Flat" view (the default). See the [Eclipse](eclipse.html#package-explorer-tip "E.2. Package Explorer View") tips section on how to toggle between the two views.

For the most part, these dependencies are straightforward and probably similar to what you're used to in your own projects. For example, there are several dependencies on Spring Framework jars, on Hibernate, DOM4J, etc.

In addition to having dependencies on a number of libraries, all lab projects also have a dependency on a common project called _rewards-common_.

![rewards-project](https://user-images.githubusercontent.com/558905/40527984-168505f8-5fbd-11e8-9122-f15d7535b94e.png)

This project is specific to Spring training courseware, and contains a number of types such as _MonetaryAmount_, _SimpleDate_, etc. You'll make use of these types throughout the course. Take a moment now to explore the contents of that jar and notice that if you double-click on the classes, the sources are available for you to browse.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>5. Configure the TODOs in STS</h2>

In the next labs, you will often be asked to work with TODO instructions. They are displayed in the `Tasks` view in Eclipse/STS. If not already displayed, click on `Window -> Show View -> Tasks` (be careful, _not `Task List`_). If you can't
see the `Tasks` view, try clicking `Other ...` and looking under `General`.

By default, you see the TODOs for all the active projects in
Eclipse/STS. To limit the TODOs for a specific project, execute the
steps summarized in the following screenshots:

![task1](https://user-images.githubusercontent.com/558905/40527969-15d132bc-5fbd-11e8-842c-3ae98ab05e2f.png)

>**Caution:** It is possible, you might not be able to see the TODOs defined within the XML files. In this case, you can check the configuration in `Preferences -> General -> Editors -> Structured Text Editor -> Task Tags` pane. Make sure `Enable searching for Task Tags` is selected.

![task2](https://user-images.githubusercontent.com/558905/40527970-15df9046-5fbd-11e8-8ace-9a59267b7244.png)

On the `Filters` tab, verify if XML content type is selected. In case of refresh issues, you may have to uncheck it and then check it again.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>6. Understanding the 'Reward Network' Application Domain and API</h2>

Before you begin to use Spring to configure an application, the pieces of the application must be understood. If you haven't
already done so, take a moment to review [Reward Dining: The Course Reference Domain](reference-domain.html "Reward Dining: The Course Reference Domain") in the preface to this Lab Guide.

This overview will guide you through understanding the background of the Reward Network application domain and thus provide context for the rest of the course.

The rewards application consists of several pieces that work together to reward accounts for dining at restaurants. In this lab, most of these pieces have been implemented for you. However, the central piece, the `RewardNetwork`, has not.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>. Review the `RewardNetwork` implementation class</h2>

The `RewardNetwork` is responsible for carrying out `rewardAccountFor(Dining)` operations. In this step you'll be working in a class that implements this interface. See the implementation class below:

![rewardnetwork-classdiagram-basic](https://user-images.githubusercontent.com/558905/40527981-16605474-5fbd-11e8-95fb-2da305b58343.png)

Take a look at your `spring-intro` project in STS. Navigate into the `src/main/java` source folder and you'll see
the root `rewards` package. Within that package you'll find the `RewardNetwork` Java interface definition:

![rewards-package](https://user-images.githubusercontent.com/558905/40527983-167792e2-5fbd-11e8-9a88-ec15dda4f301.png)

The classes inside the root `rewards` package fully define the public interface for the application, with `RewardNetwork` being the central element. Open `RewardNetwork.java` and review it.

Now expand the `rewards.internal` package and open the implementation class `RewardNetworkImpl.java`.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>8. Review the `RewardNetworkImpl` configuration logic</h2>

`RewardNetworkImpl` should rely on three supporting data access services called 'Repositories' to do
its job. These include:

1\. An `AccountRepository` to load `Account` objects to make benefit contributions to.
2\. A `RestaurantRepository` to load `Restaurant` objects to calculate how much benefit to reward an account for dining.
3\. A `RewardRepository` to track confirmed reward transactions for accounting and reporting purposes.

This relationship is shown graphically below:

![rewardnetworkimpl-classdiagram](https://user-images.githubusercontent.com/558905/40527982-166b987a-5fbd-11e8-888b-151aa57c9cfa.png)

Locate the single constructor and notice all three dependencies are injected when the `RewardNetworkImpl` is constructed.


<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>9. Implement the `RewardNetworkImpl` application logic</h2>

In this step you'll implement the application logic necessary to complete a `rewardAccountFor(Dining)` operation, delegating to your dependents as you go.

Start by reviewing your existing RewardNetworkImpl `rewardAccountFor(Dining)` implementation.
As you will see, it doesn't do anything at the moment.

Inside the task view in Eclipse/STS, complete all the TODOs.
Implement them as shown in 

[Figure 1.10](spring-intro-lab.html#fig-reward-account-for-dining-sequence-si "Figure 1.10. The  RewardNetworkImpl.rewardAccountFor(Dining)  sequence")

![rewardaccountfordining-sequence-color](https://user-images.githubusercontent.com/558905/40527979-1654a96c-5fbd-11e8-8a24-4d5b7d1dfacd.png)

>**Tip:** Use Eclipse's [autocomplete](eclipse.html#field-autocomplete-tip "E.4. Field Auto-Completion") to help you as you define each method call and variable assignment. You should not need to use operator `new` in your code. Everything you need is returned by the methods you use. The interaction diagram doesn't show what each call returns, but most of them return something. You get the credit card and merchant numbers from the `Dining`object.



<!--STEP-->

<img src="https://user-images.githubusercontent.com/558905/40613898-7a6c70d6-624e-11e8-9178-7bde851ac7bd.png" align="left" width="50" height="50" title="ToDo Logo" />
<h2>10. Unit test the `RewardNetworkImpl` application logic</h2>

How do you know the application logic you just wrote actually works? You don't, not without a test that proves it. In this step you'll review and run an automated JUnit test to verify what you just coded is correct.

Navigate into the `src/test/java` source folder and you'll see the root `rewards` package. All tests for the rewards application reside within this tree at the same level as the source they exercise. Drill down into the `rewards.internal` package and you'll see `RewardNetworkImplTests`, the JUnit test for your `RewardNetworkImpl` class.

![test-tree](https://user-images.githubusercontent.com/558905/40527972-15fa0750-5fbd-11e8-8f24-69301667de41.png)

Inside `RewardNetworkImplTests` you can notice that in the `setUp()` method, 'stub' repositories have been created and injected into the `RewardNetworkImpl` class using the constructor.

>**Note:**All the tests in this course use JUnit 5 instead of JUnit 4. Hence the `setUp()`method is annotated with `@BeforeEach`instead of `@Before`. An overview of JUnit 5 is in this [Appendix C, ***JUnit 5***](appendix_junit5.html "Appendix C. JUnit 5").

Review the only test method in the class. It calls `rewardNetwork.rewardAccountFor(Dining)` and then makes assert statements to evaluate the result of calling this method. In this way the unit test is able to construct an instance of RewardNetworkImpl using the mock objects as dependencies and verify that the logic you implemented functions as expected.

Once you reviewed the test logic, run the test. To run, right-click on `RewardNetworkImplTests` and select _Run As -> JUnit Test_.

### Results

When you have the green bar, congratulations! You've completed this lab. You have just developed and unit tested a component of a realistic Java application, exercising application behavior successfully in a test environment inside your IDE. You used stubs to test your application logic in isolation, without involving external dependencies such as a database or Spring. And your application logic is clean and decoupled from infrastructureAPIs.

You are finished!
