
# Clean Code in C++

## Table of Contents

1. [Introduction](#introduction)
2. [Variables](#variables)
3. [Functions](#functions)
4. [Classes](#classes)
5. [SOLID Principles](#solid-principles)
6. [Testing](#testing)
7. [Error Handling](#error-handling)
8. [Formatting](#formatting)
9. [Comments](#comments)
10. [Translation](#translation)

## Introduction

![Humorous image of software quality estimation as a count of how many expletives you shout when reading code](https://www.osnews.com/images/comics/wtfm.jpg)

This guide presents software engineering principles adapted from Robert C. Martin's book [_Clean Code_](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882), specifically tailored for C++. It is not a style guide; rather, it serves as a roadmap for producing [readable, reusable, and maintainable](https://github.com/ryanmcdermott/3rs-of-software-architecture) software in C++.

Not every principle outlined here must be strictly adhered to, and many may not be universally agreed upon. These are guidelines based on years of collective experience by the authors of _Clean Code_.

Our craft of software engineering is relatively young, just over 50 years old, and we are continuously evolving. When software architecture matures as much as traditional architecture, perhaps we will have more rigid rules to follow. Until then, let these guidelines be a touchstone for assessing the quality of the C++ code you and your team produce.

One more thing: understanding these principles won't instantly transform you into a better software developer, and even years of practice won't eliminate mistakes. Every piece of code begins as a rough draft, akin to wet clay waiting to be shaped into its final form. We refine it through peer review, chipping away at imperfections. Instead of criticizing your first drafts, focus on improving the code itself!

## **Variables**

### Use meaningful and pronounceable variable names

**Bad:**
```cpp
int a; // What does 'a' represent?
```

**Good:**
```cpp
int age; // Clearly indicates the variable's purpose.
```

### Use the same vocabulary for the same type of variable

**Bad:**
```cpp
getUserInfo();
getClientData();
getCustomerRecord();
```

**Good:**
```cpp
getUser(); // Consistent naming improves readability.
```

### Avoid Mental Mapping

**Bad:**
```cpp
std::vector<std::string> cities = {"Austin", "New York", "San Francisco"};
for (auto c : cities) {
    // Wait, what is `c` for again?
    processCity(c);
}
```

**Good:**
```cpp
std::vector<std::string> cities = {"Austin", "New York", "San Francisco"};
for (auto city : cities) {
    processCity(city); // Clearer context for the variable.
}
```

### Use constants for magic numbers

**Bad:**
```cpp
double area = 3.14 * radius * radius; // What does 3.14 represent?
```

**Good:**
```cpp
const double PI = 3.14;
double area = PI * radius * radius; // Clearer meaning.
```

## **Functions**

### Function arguments (2 or fewer ideally)

**Bad:**
```cpp
void createMenu(std::string title, std::string body, std::string buttonText, bool cancellable);
```

**Good:**
```cpp
struct MenuConfig {
    std::string title;
    std::string body;
    std::string buttonText;
    bool cancellable;
};

void createMenu(const MenuConfig& config);
```

### Functions should do one thing

**Bad:**
```cpp
void emailClients(std::vector<Client>& clients) {
    for (auto& client : clients) {
        if (client.isActive()) {
            email(client);
        }
    }
}
```

**Good:**
```cpp
void emailActiveClients(std::vector<Client>& clients) {
    for (const auto& client : clients) {
        if (isActiveClient(client)) {
            email(client);
        }
    }
}

bool isActiveClient(const Client& client) {
    return client.isActive();
}
```

### Function names should clearly indicate what they do

**Bad:**
```cpp
void doStuff(int x); // What does this function do?
```

**Good:**
```cpp
void calculateInterest(double principal, double rate, int time); // Clear purpose.
```

### Avoid side effects

**Bad:**
```cpp
int count = 0;

void incrementCount() {
    count++; // Modifies global state
}
```

**Good:**
```cpp
int increment(int count) {
    return count + 1; // Pure function, no side effects.
}
```

## **Classes**

### Prefer using C++11/14/17 features

**Bad:**
```cpp
class Animal {
public:
    void move();
};
```

**Good:**
```cpp
class Animal {
public:
    virtual void move() = 0; // Pure virtual function
};
```

### Use method chaining

**Bad:**
```cpp
class Car {
public:
    void setMake(std::string make);
    void setModel(std::string model);
    void setColor(std::string color);
};
```

**Good:**
```cpp
class Car {
public:
    Car& setMake(std::string make);
    Car& setModel(std::string model);
    Car& setColor(std::string color);
};

Car myCar;
myCar.setMake("Ford").setModel("F-150").setColor("red");
```

### Prefer composition over inheritance

**Bad:**
```cpp
class Engine {
public:
    void start();
};

class Car : public Engine {
public:
    void drive();
};
```

**Good:**
```cpp
class Engine {
public:
    void start();
};

class Car {
private:
    Engine engine; // Composition
public:
    void drive();
};
```

## **SOLID Principles**

### Single Responsibility Principle (SRP)

**Bad:**
```cpp
class UserSettings {
public:
    void changeSettings();
    void verifyCredentials();
};
```

**Good:**
```cpp
class UserAuth {
public:
    void verifyCredentials();
};

class UserSettings {
public:
    void changeSettings(UserAuth& auth);
};
```

### Open/Closed Principle (OCP)

**Bad:**
```cpp
class Shape {
public:
    virtual double area();
};

class Rectangle : public Shape {
public:
    double area() { return width * height; }
};
```

**Good:**
```cpp
class Shape {
public:
    virtual double area() const = 0; // Abstract
};

class Rectangle : public Shape {
public:
    double area() const override { return width * height; }
};
```

### Liskov Substitution Principle (LSP)

**Bad:**
```cpp
class Rectangle {
public:
    void setWidth(int width);
    void setHeight(int height);
};

class Square : public Rectangle {
public:
    void setWidth(int width) { Rectangle::setWidth(width); setHeight(width); }
    void setHeight(int height) { Rectangle::setHeight(height); setWidth(height); }
};
```

**Good:**
```cpp
class Shape {
public:
    virtual double area() const = 0;
};

class Rectangle : public Shape {
public:
    Rectangle(int width, int height) : width(width), height(height) {}
    double area() const override { return width * height; }
private:
    int width, height;
};

class Square : public Shape {
public:
    Square(int length) : length(length) {}
    double area() const override { return length * length; }
private:
    int length;
};
```

## **Testing**

Testing is crucial for maintaining code quality. Aim for high coverage and write tests for each new feature.

### Single concept per test

**Bad:**
```cpp
TEST(MomentTest, HandlesDateBoundaries) {
    // Test multiple concepts in one test case
}
```

**Good:**
```cpp
TEST(MomentTest, Handles30DayMonths) {
    // Test one concept
}

TEST(MomentTest, HandlesLeapYear) {
    // Test another concept
}
```

## **Error Handling**

### Don't ignore caught errors

**Bad:**
```cpp
try {
    riskyFunction();
} catch (const std::exception& e) {
    // Ignore error
}
```

**Good:**
```cpp
try {
    riskyFunction();
} catch (const std::exception& e) {
    std::cerr << "Error: " << e.what() << std::endl;
}
```

### Don't ignore rejected promises (if using async)

**Bad:**
```cpp
asyncFunction()
    .then(result => {
        // Process result
    })
    .catch(error => {
        // Ignore error
    });
```

**Good:**
```cpp
asyncFunction()
    .then(result => {
        // Process result
    })
    .catch(error => {
        std::cerr << "Error: " << error.message() << std::endl;
    });
```

## **Formatting**

### Use consistent naming conventions

**Bad:**
```cpp
int DaysInWeek = 7;
```

**Good:**
```cpp
const int daysInWeek = 7;
```

### Keep function callers and callees close

**Bad:**
```cpp
class PerformanceReview {
public:
    void perfReview();
    void getPeerReviews();
    void lookupPeers();
};
```

**Good:**
```cpp
class PerformanceReview {
public:
    void perfReview() {
        getPeerReviews();
        // Additional logic...
    }

    void getPeerReviews() {
        const auto peers = lookupPeers();
        // Process peers...
    }

    auto lookupPeers() {
        // Lookup logic...
    }
};
```

## **Comments**

### Only comment complex business logic

**Bad:**
```cpp
void hashIt(std::string data) {
    // The hash
    int hash = 0;
    // Length of string
    int length = data.length();
    // Loop through every character in data
    for (int i = 0; i < length; i++) {
        // Get character code.
        int charCode = data[i];
        // Make the hash
        hash = (hash << 5) - hash + charCode;
        // Convert to 32-bit integer
        hash &= hash;
    }
}
```

**Good:**
```cpp
void hashIt(std::string data) {
    int hash = 0;
    int length = data.length();

    for (int i = 0; i < length; i++) {
        int charCode = data[i];
        hash = (hash << 5) - hash + charCode; // Update hash with character code
    }
}
```

### Avoid leaving commented-out code

**Bad:**
```cpp
// void unusedFunction() {}
```

**Good:**
```cpp
// Function removed; see version history.
```

### Avoid journal comments

**Bad:**
```cpp

void combine(int a, int b) {
    return a + b;
}
```

**Good:**
```cpp
void combine(int a, int b) {
    return a + b; // Implementation updated for clarity
}
```

### Avoid positional markers

**Bad:**
```cpp
$scope.model = {
    menu: "foo",
    nav: "bar"
};
```

**Good:**
```cpp
$scope.model = {
    menu: "foo",
    nav: "bar"
}; // Instantiation of model
```

## **Translation**

This guide can be translated into other languages. Contributions are welcome!

---

By following these guidelines, you can write clean, maintainable, and efficient C++ code. Remember, the goal is not only to write code that works but to write code that others can read and understand.
