+++
title = "Projects"
+++

These are some of the projects I'm working on...

## Qadenz

Qadenz is a robust and somewhat opinionated test automation library written in Java. In addition to wrapping Selenium element-interaction functionality and TestNG suite configuration, I've implemented custom features for Qadenz, such as a Condition & Expectation based API that powers both the fluent waits and assertion functionality, as well as a detailed and easy to read HTML reporting component that includes integrated screenshots.

The UI Modeling strategy I encourage with the use of Qadenz is centric to the Locator object, which represents a mapped UI element. This ultimately allows teams to use a simplified Page Object Model, with the ability to move past antiquated patterns such as the `PageFactory` and `@FindBy` annotated element mappings. Combined with the Commands-style wrapped Selenium functions, teams can also avoid the added maintenance overhead (nightmares) involved with using extra abstraction layers like Gherkin/Cucumber.

Knowing that each team and each testing project will have their own technical needs, I've designed Qadenz to be completely extensible, so that custom configuration and functionality can be implemented without having to make changes to the underlying code-base. This encourages importing the library as a dependency and extending components where necessary, as opposed to the maintainence overhead (and upgrading issues) encountered by forking the project and changing the core code.

Qadenz has had several names and countless revisions and improvements in the last 10 years. This has been a continually maintained personal code-base that has served as a reference design for the tools I build at my jobs. At the encouragement of several friends and colleagues, I decided in 2019 to evolve the project once again by rebuilding the project from scratch, finally tackling some long standing challenges, and ultimately releasing the library to the testing community.

The code is available on [GitHub](https://github.com/qadenz/qadenz), and documentation is at [qadenz.dev](https://qadenz.dev).

Qadenz is released under the [PolyForm Internal Use License 1.0](https://polyformproject.org/licenses/internal-use/1.0.0).

## WebDriver Sizzle Injector

I'm a huge fan of CSS Selectors and I use them exclusively in my automation projects. Adding the capabilities of SizzleJS into the mix makes CSS Selectors even better with a number of avenues for additional parameterization.

This project, as implied by the name, injects SizzleJS onto the DOM of the UI under test. This introduces handy pseudo-classes for selector expressions such as `:contains()` for matching inner text, `:eq()` for instance matching, and various relative-position options.

This began as a means to move past using XPath selectors in my Page Objects, and was originally based on an example I found in a blog post that has long since disappeared. As I became more familiar with the components in the Selenium library concerning element initialization (specifically the `By` class and the various selector type objects), I refined the design of the injector to follow more closely the design patterns set forth in Selenium. Initializing a `WebElement` using a Sizzle selector is as simple as passing a slightly different argument to the `WebDriver`:

```java
    driver.findElement(BySizzle.css("#loginForm
        .button:contains('sign in')"));
```

The Sizzle Injector is available on [GitHub](https://github.com/tim-slifer/webdriver-sizzle-injector), and is licensed under the [MIT License](https://github.com/tim-slifer/webdriver-sizzle-injector/blob/master/LICENSE).

## Superfly-CSV

I love my Excel sheets as much as the next nerd, but one of my early lessons in automated testing was, don't use Excel sheets. Apart from libraries that read Excel sheets being rather cumbersome in my opinion, the binary nature of Excel files makes working with them (successfully, without problems) in a collaborative environment such as Git all but impossible. I found it absolutely infuriating when teammates and myself would inadvertently commit over each other's updates to Excel files, losing many hours of work.

As such, I prefer to use CSV files for tracking organized data sets.

Superfly-CSV creates a clean API that allows one to traverse rows of a CSV file, pull values from "columns" by name, and even filter/exclude rows based on column values. It's a handy solution that avoids the pitfalls of Excel sheets, and yields cleaner code than what would be encountered with Apache POI.

Superfly-CSV is available on [GitHub](https://github.com/tim-slifer/superfly-csv), and is licensed under the [MIT License](https://github.com/tim-slifer/superfly-csv/blob/master/LICENSE).

## AfterDark IntelliJ Editor Theme

When I started coding, I found myself quickly bored of the default editor themes in Eclipse and later in IntelliJ. There were many options on the web for downloadable themes, and I settled into a small group of options that I retated through as my tastes and moods changed. Eventually, it occurred to me that a color theme could be a functional part of my development tool rather than strictly a cosmetic change.

I started with a dark theme and a pallette of neon colors. The resulting effort yielded a theme where every option has been deliberately chosen in order to maximize the readability of code. Each color represents a specific type of expression and text styling represents a specific type of usage. By taking visual cues from how the theme presents code in my editor, I can more quickly understand the structure of the code I'm reading.

I eventually plan to build this into a plugin and publish to the JetBrains Marketplace, but for now it exists as a raw `.icls` file that can be imported into IntellJ.

AfterDark is available on [GitHub](https://github.com/tim-slifer/after-dark-intellij).

## IntelliJ Codestyle

I do like to keep my code clean and well-formatted. It's also absurdly convenient to have my IDE take care of this task for me, rather than me having to worry about whether I left extra spacing between operators, if I left too many empty lines, or if I exceeded my line width.

This one isn't overly restrictive and it generally stays out of the way. The intent is to enforce some basic rules like line width and tab style (spaces over tabs FTW). The rest of the settings are mostly just for consistency. I've found this extremely handy in a team environment for keeping code nice and tidy.

I usually only update this when JetBrains releases a major version of IntelliJ, which equates to about once a year.

The formatter is avaialble on [GitHub](https://github.com/tim-slifer/intellij-codestyle).
