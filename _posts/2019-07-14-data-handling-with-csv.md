---
layout: post
title: Data handling with CSV for fun and profit
author: Tim Slifer
excerpt_separator: <!--more-->
---

Ok, maybe handling data with CSV files may neither be fun nor profitable. In fact, I'd even opine that handling data 
in general is probably one of the more unpleasant parts of designing an automation solution. Today, I'll share some of 
my own experiences and a tool I've just open sourced.

<!--more-->

There are a multitude of approaches to working with test data in the world of automation. Some have certainly 
been more versatile than others, generally speaking. In my experience there never has been a silver bullet solution 
that renders all the others useless. Easily the most common approach I've encountered being discussed by colleagues and 
candidates I've interviewed is Excel spreadsheets.

# Excel, our old friend

Spreadsheets are an obvious choice with the familiar layout of rows and columns, and the general ubiquity of the 
application itself. In my early days of manual testing, we used spreadsheets extensively to not only track data, but 
metrics and even tests themselves (when we didn't have a proper test case manager). The problems come, however, we 
when bring this concept into a more development-oriented environment.

The first, largest, and in my experience, often ignored, issue with Excel sheets is source control. Since Excel files
are binary, Git and SVN can't track individual changes without secondary tools that "decompile" the sheet into XML 
first. Without being able to diff the contents of the file, there is no way to track who made what changes and when. 
Additionally, working with a binary file in source control can lead to much lost work due to commits that overwrite 
the file. If one person commits their changes to a sheet before another (and their changes aren't sync'd somehow), 
their changes will be lost when the next person commits their changes.

Beyond file management, though, Excel gives us a lot of functionality that can get us, as testers, (in my opinion) into 
trouble. Excel formulas are very powerful and extremely handy when working with stores of data. In the realm of 
testing, however, I've seen formulas added to spreadsheets, for example, to generate the results of calculations that
are performed within the application under test. By comparing the system output to Excel formula output, are we 
testing the system against expected values, or that Excel gives us the same results as the application?

In order to use these spreadsheets, we have libraries like Apache POI. Modeling and working with an Excel
sheet requires a lot of code for even simple data. Once we start getting into formulas, formatting, and separate 
tabs, we start to get to that point where getting the spreadsheet handler in working form is as big a task is 
building the rest of the test project.

# Enter the alternative

The format I always recommend when it comes to sheet-based data is CSV. CSV allows for a usage scenario that is 
very similar to that of Excel, while avoiding the unnecessary complexity in formatting, formulas, or tabs. 
Personally, I believe the simplicity of the CSV format allows us to keep out data from getting unnecessarily complex.
Since we don't have the ability to mess around with complex formulas, any conditional formatting, or files with 
multitudes of tabs, we can keep our focus narrow and our approach direct when it comes to managing our data values.

On teams where we've opted for CSV over Excel, we've been able to easily achieve a data handling approach that is 
very lightweight, and most importantly, easy to use. Since CSV is text based, our source control issues disappeared 
nearly instantaneously, and we had total insight into the contents of our data files to see what changes were committed,
when, and by whom. In a number of instances, we had tests that were suddenly failing despite no code changes in the 
associated area of the application. By having the ability to easily track our changes, we quickly identified altered 
data that was the source of the failed test, not a defect in the system.

Being a far more open format than Excel, there are [plenty of options](http://csveed.org/comparison.html) in terms of 
library support. I tend to gravitate toward the easy-to-use OpenCSV library, though I always would recommend that 
anyone purusing a tool such as this to do their own research and experimentation on the various tools available.

# Superfly-CSV

[Superfly-CSV](https://github.com/tim-slifer/superfly-csv) is a wrapper for OpenCSV that I've built and recently 
open-sourced, and is available in [Maven Central](https://search.maven.org/search?q=a:superfly-csv). Many of the teams 
with whom I've worked have implemented CSV handlers in their projects, from bare-boned to very full-feature. This 
project represents the lessons learned, the results of both good and bad ideas, and some neat ideas I've encountered 
along the way. The design of many of these tools has been very collaborative, so I took the opportunity on this 
project to approach with an "if I had to do it my own way" mentality.

There are two main components in this library, the `CsvLoader`, and the `CsvFile`. The `CsvLoader` is called 
statically, whenever a new file is to be loaded. The only required argument is the path to the file (relative to the 
classpath) and the name of the file itself. The `CsvFile` represents the virtualized CSV data, along with the 
navigation and data retrieval functions.

Specific usage instructions are available in the project's 
[README](https://github.com/tim-slifer/superfly-csv/blob/master/README.md) file, so I'll focus here on implementation.

An extremely simple example is tracking credentials for test user accounts in an application. Let's consider a 
scenario where we have Administrator and System User roles, with separate usernames and passwords for each. A simple 
CSV layout could be:

```csv
role,username,password
user,user1name,p@55w0rD1
admin,admin1name,Pa55WorD123
```

In our test case, we would create a `CsvFile` instance, filter to the Role we need, call the appropriate values, and 
pass to our Page Objects.

```java
@Test
public void testUserLoginSuccessful() {
    
    // First, load the file and filter based on the value
    // of the first column.
    CsvFile users = CsvLoader.load("users.csv");
    csv.filter("admin");
    
    // Pass the CSV values to the Page Objects, which in turn
    // input the values into the fields on the UI.
    LoginPage loginPage = new LoginPage();
    loginPage.enterUsername(csv.valueOf("username"));
    loginPage.enterPassword(csv.valueOf("password"));
    loginPage.clickSignIn();
    
    // Subsequent code will validate successfull authentication.
}
```

Working with multiple CSV files is as simple as creating new instances of the `CsvFile` object. Filter and Exclude 
operations can be performed repeatedly as long as data remains on the file. `CsvFile` instances can even be cloned, 
which creates save-points in the progression of working with a file. Navigating rows is as simple as calling 
`setNextRow()`, or `setPreviousRow()`, or skipping directly to a specific row by `setCurrentRow(int row)`. Full 
instructions are available on the [README](https://github.com/tim-slifer/superfly-csv/blob/master/README.md) file in 
the project.

I've released Superfly-CSV under the [MIT License](https://opensource.org/licenses/MIT).

> I'd certainly welcome any feedback from anyone that tries out this library. Drop me an [email](contact@slifer.io) and
> let me know what you think.
