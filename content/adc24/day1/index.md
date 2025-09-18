+++
date = '2025-09-15T21:09:26-03:00'
draft = false
title = 'Advent of Code 24 Day 1'
tags = ["AdventOfCode24", "Programming", "C++"]
quickSummary = "First puzzle! Historian Hysteria"
+++

[Link to the github repository](https://github.com/gabrielholzmann/AdventOfCode24)

Hello, welcome to the first post on my series of solutions to advent of code 2024!

Here are some of the characteristics you will find in my solutions:
- All of my puzzles will be solved in C/C++
- I will not provide my personal input
- If you have any suggestions, comments or optimizations to my code please [contact me](/contact)

First of all get acquainted to the puzzle in this [link](https://adventofcode.com/2024/day/1)

# Part 1

We started with a pretty simple puzzle here are the steps 
1. read the left and right part into lists
2. sort both lists
3. calculate the difference between each element and add all those differences

You can see these steps in the code below, let's explore further how each part works:

```cpp
int main() {
    std::ifstream file = openFile("../input.txt");

    lists locationList = readLists(file);

    sortLists(locationList);

    std::cout << calculateDistances(locationList) << std::endl;

    file.close();
    return 0;
}
```

## File Opening

Using std::ifstream we can open a file and return it, we also check for errors if the file doesn't exist for example

```cpp
std::ifstream openFile(const std::string &fileName) {
    std::ifstream file;
    file.open(fileName, std::ifstream::in);

    if (!file.is_open()) {
        std::cerr << "Error opening file " << fileName << std::endl;
        throw std::runtime_error("Error opening file");
    }

    return file;
}
```

## Reading the file

Here we create a struct with 2 vectors that will hold the left and right list

To actually read the input we just have a while loop and the >> operator reading numbers into 2 variables and them pushing them onto our struct

```cpp
typedef struct lists {
    std::vector<int> leftList;
    std::vector<int> rightList;
} lists;

lists readLists(std::ifstream &file) {
    lists locationLists;

    int leftNumber = 0, rightNumber = 0;

    while (file >> leftNumber >> rightNumber) {
        locationLists.leftList.push_back(leftNumber);
        locationLists.rightList.push_back(rightNumber);
    }

    return locationLists;
}
```

## Sort the list

We just use std::sort

```cpp
void sortLists(lists &locationLists) {
    std::sort(locationLists.leftList.begin(), locationLists.leftList.end());
    std::sort(locationLists.rightList.begin(), locationLists.rightList.end());
}
```

## Calculate distance

Here we go element by element calculate the absolute difference between them and add it to the total

```cpp
int calculateDistances(const lists &locationList) {
    int totalDistance = 0;

    for (int i = 0; i < locationList.leftList.size(); i++) {
        totalDistance += std::abs(locationList.rightList[i] - locationList.leftList[i]);
    }

    return totalDistance;
}
```

# Part 2

Part 2 is a little more complicated we need to:
1. figure out how often each number appears in the right list(a frequency map).  For that we will use an unordered_map(hash map)
2. loop through the left list and multiply it by how many times it appears in the right list(frequency map) and add them all up

\*Everything else not mentioned here is the same as part 1
```cpp
int main() {
    std::ifstream file = openFile("../input.txt");

    lists locationList = readLists(file);

    frequencyMap fm = calculateFrequency(locationList);

    std::cout << calculateSimilarityScore(locationList, fm) << std::endl;

    file.close();
    return 0;
}
```
## Calculating right list frequency

Loop through the right list and add 1 to it on the hash map to calculate how many times it appears on the right list

```cpp
frequencyMap calculateFrequency(const lists &locationLists) {
    frequencyMap fm;

    for (const auto &locationNumber : locationLists.rightList) {
        fm[locationNumber] += 1;
    }

    return fm;
}
```

## Similarity score

Loop through the left list and multiply by its frequency on the frequency map(if we can find it) and them add it all up

```cpp
int calculateSimilarityScore(const lists &locationList, const frequencyMap &fm) {
    int similarityScore = 0;

    for (const auto &locationNumber : locationList.leftList) {
        if (auto iter = fm.find(locationNumber); iter != fm.end())
            similarityScore += locationNumber * iter->second;
    }

    return similarityScore;
}
```