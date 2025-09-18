+++
date = '2025-09-16T21:39:28-03:00'
draft = false
title = 'Advent of Code 24 Day 2'
tags = ["AdventOfCode24", "Programming", "C++"]
quickSummary = "Second puzzle! Let's check a few reports!"
+++

[Link to the github repository](https://github.com/gabrielholzmann/AdventOfCode24)


Here are some of the characteristics you will find in my solutions:
- All of my puzzles will be solved in C/C++
- I will not provide my personal input
- If you have any suggestions, comments or optimizations to my code please [contact me](/contact)

First of all get acquainted to the puzzle in this [link](https://adventofcode.com/2024/day/2)

# Part 1

## Main

Our **main** consists of:
1. opening a file
2. reading the reports
3. looping through all the reports and checking if they are valid

```cpp
int main() {
    FILE *fp = openFile("../input.txt");

    ReportLists rl = readReportLists(fp);

    std::cout << calculateValidReportsTotal(rl) << std::endl;

    return 0;
}
```

## Opening files

This time we open the file and if it's not valid we throw a std::runtime_error

```cpp
FILE *openFile(const char *filename) {
    FILE *fp = fopen(filename, "r");

    if (!fp) {
        throw std::runtime_error("Could not open file");
    }

    return fp;
}
```

## Reading reports

First we have a data structure that defines a single report(Report) and another one that defines a vector of reports(ReportList)

To actually read the reports we create a line buffer and use fgets to read into it, next we use the strtok function to separate the string into tokens in this case " "(space) 
because the numbers in our input are separated by spaces.

With atoi we read that number and push it into a new report and after all numbers are read we push that report into the report list

```cpp
typedef std::vector<int> Report;

typedef struct ReportLists {
    std::vector<Report> reportLists;
} ReportLists;

ReportLists readReportLists(FILE *fp) {
    ReportLists rl;
    char line[50];

    while (fgets(line, 50, fp)) {
        Report r;

        char *ptr = strtok(line, " ");
        while (ptr) {
            r.push_back(atoi(ptr));
            ptr = strtok(nullptr, " ");
        }

        rl.reportLists.push_back(r);
    }
    return rl;
}
```

## Checking reports
**calculateValidReportsTotal()** just loop through all our reports in the report list and if it's valid it adds to the total

Now let's look at the most important part: How do we know if a report is valid?

We need 2 pieces of information:
1. is the interval increasing or decreasing
2. is the interval >= 1 and <= 3

To answer question one we can take the difference of the first and second number 
- if the first number is greater the modifier is positive
- if the second number is greater that means our difference will be negative so we need to multiply by -1 to turn it into a positive number

We can't just take the absolute value with **std::abs()** because then we would have no information if the intervals are all increasing or decreasing

So now we just loop through all the number taking the difference and multiplying by the modifier if it's out of the permitted interval we return false
```cpp
bool checkIfReportIsValid(Report &r) {
    int modifier = (r[0] > r[1]) ? 1 : -1;

    for (int i = 0; i < r.size() - 1; ++i) {
        int diff = (r[i] - r[i + 1]) * modifier;

        if (diff < 1 || diff > 3)
            return false;
    }
    return true;
}

int calculateValidReportsTotal(ReportLists &rl) {
    int totalValid = 0;

    for (auto &r : rl.reportLists) {
        totalValid += checkIfReportIsValid(r);
    }

    return totalValid;
}

```

# Part 2

Now we can tolerate 1 bad level the only difference is how we check reports.

I know it's a brute-force solution but if you have any suggestions on how to implement a better one please [contact me](/contact)

\*Everything else not mentioned here is the same as part 1

## Removing a single element

Looking at the new intermediary function **checkIfReportIsValidByRemovingOne()**

First try to see if the report is valid, if it is return true

Else loop through every element, create a temp report and remove that element from temp and check if it's valid. If it is return true

```cpp
bool checkIfReportIsValid(const Report &r) {
    int modifier = (r[0] > r[1]) ? 1 : -1;

    for (int i = 0; i < r.size() - 1; ++i) {
        int diff = (r[i] - r[i + 1]) * modifier;

        if (diff < 1 || diff > 3)
            return false;
    }
    return true;
}

bool checkIfReportIsValidByRemovingOne(const Report &r) {
    if (checkIfReportIsValid(r))
        return true;

    for (int i = 0; i < r.size(); ++i) {
        Report temp = r;
        temp.erase(temp.begin() + i);
        if (checkIfReportIsValid(temp))
            return true;
    }
    return false;
}

int calculateValidReportsTotal(ReportLists &rl) {
    int totalValid = 0;

    for (auto &r : rl.reportLists) {
        totalValid += checkIfReportIsValidByRemovingOne(r);
    }

    return totalValid;
}
```