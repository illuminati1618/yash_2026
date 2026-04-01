---
toc: true
layout: post
title: My FRQ Learning Journey
description: Teaching, solving, and reflecting on AP CSA Free Response Questions — from algorithms to reverse polish notation
permalink: /frq-learning-journey
author: Yash Parikh
categories: ['CSA', 'FRQ', 'Reflection']
---

## The Highlight: Teaching Reverse Polish Notation

By far the most exciting and memorable piece of my FRQ journey was building and teaching a **Reverse Polish Notation (RPN) calculator**. RPN (also called postfix notation) flips the script on how we normally write math — instead of `3 + 4`, you write `3 4 +`. Operators come *after* their operands, which eliminates the need for parentheses and operator precedence entirely.

This was not just a fun algorithm to implement — it was a masterclass in **stack-based computation**, one of the most important data structures in CS.

<img width="656" height="900" alt="Image" src="https://github.com/user-attachments/assets/9ac1adda-ce57-401b-8093-6228724a88af" />

### How RPN Works

| Infix (normal) | Postfix (RPN) |
|---|---|
| `3 + 4` | `3 4 +` |
| `(2 + 3) * 4` | `2 3 + 4 *` |
| `5 * (1 + 2) / 3` | `5 1 2 + * 3 /` |

**The algorithm:**
1. Read tokens left to right
2. If the token is a **number**, push it onto the stack
3. If the token is an **operator**, pop the top two values, apply the operator, push the result
4. At the end, the stack holds exactly one value — the answer

### The Java Implementation

```java
import java.util.Stack;

public class RPNCalculator {
    public static double evaluate(String expression) {
        Stack<Double> stack = new Stack<>();
        String[] tokens = expression.trim().split("\\s+");

        for (String token : tokens) {
            switch (token) {
                case "+": stack.push(stack.pop() + stack.pop()); break;
                case "-": {
                    double b = stack.pop(), a = stack.pop();
                    stack.push(a - b);
                    break;
                }
                case "*": stack.push(stack.pop() * stack.pop()); break;
                case "/": {
                    double b = stack.pop(), a = stack.pop();
                    stack.push(a / b);
                    break;
                }
                default: stack.push(Double.parseDouble(token));
            }
        }
        return stack.pop();
    }

    public static void main(String[] args) {
        System.out.println(evaluate("3 4 +"));       // 7.0
        System.out.println(evaluate("2 3 + 4 *"));   // 20.0
        System.out.println(evaluate("5 1 2 + * 3 /")); // 5.0
    }
}
```

### Why This Stuck With Me

- **Stacks are everywhere.** Browsers use them for back navigation. Compilers use them to evaluate expressions. Undo/redo systems use them. Building an RPN calculator makes you *feel* why.
- **Order matters.** For subtraction and division, the order you pop from the stack matters — a subtle bug I had to debug and understand precisely.
- **It's elegant.** No parentheses. No precedence rules. Just a stack and a scan. The simplicity is beautiful.

Teaching this to classmates made it click even more. Having to explain *why* you pop `b` before `a` for subtraction, or why the stack always has exactly one element at the end, pushed my understanding from "I can code it" to "I truly get it."

---

## FRQ Homework: Issue by Issue

Over the year I tracked every FRQ attempt in GitHub issues. Here's what each one taught me.

---

### Issue #27 — 1.1 Algorithms Homework (09/30/2025)

[illuminati1618/yash_2025#27](https://github.com/illuminati1618/yash_2025/issues/27)

My very first CS homework submission of the year. This introduced the core idea that **an algorithm is a precise, finite sequence of steps** — something that sounds obvious but becomes non-trivial the moment you try to write one for a real problem.

Key takeaway: clarity and precision in step design are what separate a vague plan from an actual algorithm.

---

### Issue #29 — FRQ 2024 Question 2 (01/23/2026)

[illuminati1618/yash_2025#29](https://github.com/illuminati1618/yash_2025/issues/29)

My first real FRQ attempt of the trimester. Working through a 2024 exam question under time constraints was humbling — the problem forced me to read carefully and not jump to code before I understood the structure of what was being asked.

Key takeaway: read the entire problem before writing a single line. The helper methods the exam provides are hints, not distractions.

---

### Issue #31 — FRQ 2016 Question 3 (02/07/2026)

[illuminati1618/yash_2025#31](https://github.com/illuminati1618/yash_2025/issues/31)

A 2016 exam question that tested **string manipulation and method reuse**. This is the type of FRQ where a provided helper method is designed to be called repeatedly in Part B to solve a different problem than Part A.

Key takeaway: when the exam provides a helper method, your job in Part B is almost always to *call it creatively*, not to rewrite its logic.

---

### Issue #32 — FRQ HW 4: Interfaces and Polymorphism (02/11/2026)

[illuminati1618/yash_2025#32](https://github.com/illuminati1618/yash_2025/issues/32)

This FRQ introduced me to **interface-based design** in a real problem context. Key notes I documented:

| Concept | What I Learned |
|---|---|
| Interfaces and Polymorphism | Multiple implementations work interchangeably through an interface contract |
| Boundary Conditions | Use `>=` / `<=` carefully — exclusive vs. inclusive is a common mistake |
| Interface Abstraction | Interfaces define *what* methods exist, not *how* they work |
| Encapsulation | Instance variables stay private; access controlled through methods |
| Enhanced For-Loops | Iterating over interface types enables polymorphic traversal |
| Short-Circuit Logic | Return early once a condition is met — don't loop unnecessarily |
| Naming Conventions | Clear names make code self-documenting |

Key takeaway: interfaces let you write code that works for any future implementation — this is one of the most powerful OOP tools in Java.

---

### Issue #33 — FRQ 2024 Question 2 Revisit (02/11/2026)

[illuminati1618/yash_2025#33](https://github.com/illuminati1618/yash_2025/issues/33)

A second pass at the 2024 Q2 problem, this time with more confidence. Revisiting the same FRQ after a few weeks revealed gaps I didn't know I had the first time. What felt hard in January felt manageable in February — proof that distributed practice works.

Key takeaway: returning to an FRQ you already "solved" is one of the highest-value study activities. You almost always find something you missed.

---

### Issue #35 — FRQ 2017 Q1: Digits (03/14/2026)

[illuminati1618/yash_2025#35](https://github.com/illuminati1618/yash_2025/issues/35)

The Digits FRQ required building a class that stores individual digits of an integer in an `ArrayList` and checks whether they form a strictly increasing sequence. My documented takeaways:

- **Constructor naming** — must match the class name exactly in Java
- **Digit extraction** — use `num % 10` to get the last digit, `num /= 10` to strip it, insert at index 0 to preserve order
- **Zero edge case** — `0` must be handled separately since `while (num != 0)` never executes for it
- **Strictly increasing check** — loop through adjacent pairs; return `false` immediately on the first violation
- **Loop bound** — use `i < digitList.size() - 1` to avoid `IndexOutOfBoundsException`
- **Single-digit numbers** — automatically satisfy strictly increasing (no adjacent pair to violate)

```java
public Digits(int num) {
    digitList = new ArrayList<>();
    if (num == 0) {
        digitList.add(0);
        return;
    }
    while (num != 0) {
        digitList.add(0, num % 10);
        num /= 10;
    }
}

public boolean isStrictlyIncreasing() {
    for (int i = 0; i < digitList.size() - 1; i++) {
        if (digitList.get(i) >= digitList.get(i + 1)) return false;
    }
    return true;
}
```

Key takeaway: the zero edge case and the `size() - 1` loop bound are the two places where real exam answers lose points. Edge cases are not afterthoughts — they're part of the solution.

---

### Open-Coding-Society/pages#637 — Teaching FRQ 2024 Q2

[Open-Coding-Society/pages#637](https://github.com/Open-Coding-Society/pages/issues/637)

This issue marks the moment I shifted from student to teacher. Submitting a teaching contribution to the Open Coding Society pages meant I had to understand a problem well enough to explain it clearly to others — which is a fundamentally different and harder skill than solving it yourself.

Key takeaway: you don't truly know something until you can teach it. The moment someone asks "but why?" you find out exactly how deep your understanding really goes.

---

## Patterns Across All FRQs

After working through every one of these, certain patterns show up over and over:

| Pattern | Where It Appeared |
|---|---|
| Helper method reuse | Issues #31, #33, Phrase/2016 FRQs |
| Edge case handling | Issue #35 (zero), Interface FRQ (boundary conditions) |
| Stack-based logic | RPN Calculator (the crown jewel) |
| Adjacent element comparison | Issue #35, WordChecker-style problems |
| Loop until condition | Phrase Part B, RPN evaluation loop |
| Return early / short-circuit | Interface FRQ, isStrictlyIncreasing |
| Index arithmetic off-by-one | Issue #35 (`size() - 1`), all substring problems |

---

## What This Year of FRQs Taught Me

**Solving an FRQ teaches you the algorithm. Teaching one teaches you why it works.**

The RPN calculator was the most memorable because it made a fundamental CS concept — stacks — tangible and satisfying. Every other FRQ built a piece of the picture: string manipulation, 2D arrays, ArrayList design, interface contracts, digit arithmetic. Together they map to real software engineering problems.

Going into AP exams and beyond, these are the habits I'm keeping:

1. Read the entire problem and understand the provided methods before writing anything
2. Handle edge cases as part of the design, not as an afterthought
3. When a helper method is given, your job is to use it creatively
4. Teaching is the highest form of studying
5. A stack solves more problems than you'd think

---

*Tracked via [yash_2026 CSA GitHub Issues](https://github.com/users/illuminati1618/projects/8)*
