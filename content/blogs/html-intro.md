---
title: Understanding TypeScript: Building a Fitness App
slug: intro-to-html
description: Understanding TypeScript via project building
longDescription: Understanding TypeScript via project building
cardImage: "https://zaggonaut.dev/michael-dam-unsplash.webp"
tags: ["code", "html"]
readTime: 15
featured: true
timestamp: 2024-12-18T02:39:03+00:00
---

# Understanding TypeScript: Building a Fitness App

## Introduction

Building a full-stack application is one of the best ways to learn TypeScript. In this project walkthrough, I explored how TypeScript enhances a React fitness application, focusing on the practical aspects that make TypeScript valuable for real-world development.

## Understanding TypeScript Fundamentals

### The Type System Basics

TypeScript's power comes from its ability to catch errors before runtime. Let's start with the fundamentals:

```typescript
// Basic type annotations
const selectedPage: string = "home";
const isMenuToggled: boolean = false;
const userAge: number = 25;
```

In practice, TypeScript often uses **type inference**, meaning it can figure out types automatically:

```typescript
const [selectedPage, setSelectedPage] = useState("home");
// TypeScript infers selectedPage is a string
```

However, being explicit helps when types are complex or unclear:

```typescript
const [selectedPage, setSelectedPage] = useState<string>("home");
// Now it's crystal clear this is a string state
```

## Enums: Better Than String Constants

One of the most useful TypeScript features for navigation is **enums**. Instead of using plain strings that can have typos:

```typescript
// Error-prone approach
setSelectedPage("home");
setSelectedPage("homee"); // Typo! No error caught
```

We can use enums:

```typescript
enum SelectedPage {
  Home = "home",
  Benefits = "benefits",
  OurClasses = "ourclasses",
  ContactUs = "contactus"
}

// Type-safe approach
setSelectedPage(SelectedPage.Home);
setSelectedPage(SelectedPage.Homee); // TypeScript error!
```

Enums are special because they exist at both compile-time (for type checking) and runtime (as actual values you can use in your code). This makes them perfect for navigation states, dropdown options, and other fixed sets of values.

## Typing React Components

### Props and Component Signatures

Every React component with props needs a type definition:

```typescript
type Props = {
  page: string;
  selectedPage: SelectedPage;
  setSelectedPage: (value: SelectedPage) => void;
};

const Link = ({ page, selectedPage, setSelectedPage }: Props) => {
  // Component logic
};
```

Breaking this down:

- `page: string` - A simple string prop
- `selectedPage: SelectedPage` - Must be one of our enum values
- `setSelectedPage: (value: SelectedPage) => void` - A function that takes a SelectedPage and returns nothing (void)

### Optional Props

Sometimes props aren't always needed:

```typescript
type BenefitType = {
  icon: JSX.Element;
  title: string;
  description?: string; // The ? makes this optional
  image: string;
};
```

The optional `description` can be undefined, and TypeScript won't complain when you omit it.

## Working with Arrays and Objects

### Array Typing

When you have an array of objects, TypeScript needs to know the structure:

```typescript
const benefits: Array<BenefitType> = [
  {
    icon: <HomeModernIcon className="h-6 w-6" />,
    title: "State of the Art Facilities",
    description: "Lorem ipsum dolor sit amet",
    image: image1
  },
  // More items...
];
```

This tells TypeScript: "This is an array where every item matches the BenefitType structure."

### Mapping with Types

When mapping over arrays, TypeScript knows the type of each item:

```typescript
benefits.map((benefit: BenefitType) => (
  <Benefit
    key={benefit.title}
    icon={benefit.icon}
    title={benefit.title}
    description={benefit.description}
  />
));
```

The `benefit: BenefitType` annotation is optional here (TypeScript can infer it), but it helps with IntelliSense and clarity.

## Advanced: Type Assertions

Sometimes you know more about a type than TypeScript does:

```typescript
const lowerCasePage = page.toLowerCase().replace(/ /g, "");
// TypeScript doesn't know this creates a valid SelectedPage

onClick={() => setSelectedPage(lowerCasePage as SelectedPage)}
// We tell TypeScript: "Trust me, this IS a SelectedPage"
```

The `as` keyword is a **type assertion**. Use it sparingly and only when you're certain about the type. Overusing it defeats the purpose of TypeScript.

## Interfaces vs Types

You'll see both `interface` and `type` in TypeScript code:

```typescript
// Using interface
interface BenefitType {
  icon: JSX.Element;
  title: string;
}

// Using type
type BenefitType = {
  icon: JSX.Element;
  title: string;
};
```

For most cases, they're interchangeable. The key differences:

- Interfaces can be extended and merged
- Types can represent unions and more complex types
- **Choose one approach and stay consistent** in your project

## Practical TypeScript Patterns

### 1. Shared Type Definitions

Create a central file for reusable types:

```typescript
// shared/types.ts
export enum SelectedPage {
  Home = "home",
  Benefits = "benefits",
  OurClasses = "ourclasses",
  ContactUs = "contactus"
}

export interface BenefitType {
  icon: JSX.Element;
  title: string;
  description?: string;
  image: string;
}

export interface ClassType {
  name: string;
  description?: string;
  image: string;
}
```

### 2. Function Typing

Functions that don't return values use `void`:

```typescript
type Props = {
  setSelectedPage: (value: SelectedPage) => void;
};
```

Functions with return values specify the type:

```typescript
const calculateAge = (birthYear: number): number => {
  return new Date().getFullYear() - birthYear;
};
```

### 3. React Hook Form with TypeScript

Form libraries work beautifully with TypeScript:

```typescript
const {
  register,
  trigger,
  formState: { errors }
} = useForm();

<input
  {...register("email", {
    required: true,
    pattern: /^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i,
  })}
/>

{errors.email?.type === "required" && (
  <p>This field is required.</p>
)}
```

TypeScript ensures you're accessing form properties correctly and catches typos in field names.

## Common TypeScript Pitfalls

### 1. The `any` Escape Hatch

```typescript
const handleSubmit = async (e: any) => {
  // Using 'any' turns off type checking
};
```

While sometimes necessary (like ambiguous event types), overusing `any` defeats TypeScript's purpose. Use it only when the time spent figuring out the exact type isn't worth the benefit.

### 2. Forgetting to Import Types

```typescript
import { BenefitType } from "@/shared/types"; // ✅ Import the type
```

Unlike runtime code, TypeScript types need explicit imports from their definition files.

### 3. Compile Time vs Runtime

TypeScript types don't exist in your final JavaScript bundle. They're only for development:

```typescript
// This code doesn't exist after compilation
type Props = { name: string };

// Enums are an exception - they DO exist at runtime
enum SelectedPage { Home = "home" }
```

## Why TypeScript Matters

### 1. **IntelliSense and Autocomplete**

Your editor knows what properties exist:

```typescript
benefit. // Your editor suggests: icon, title, description, image
```

### 2. **Refactoring Confidence**

Change a type definition, and TypeScript shows every place that needs updating. No more hunting through files for where you used a particular property name.

### 3. **Self-Documenting Code**

Types serve as inline documentation:

```typescript
setSelectedPage: (value: SelectedPage) => void
// Anyone reading this knows exactly what it expects
```

### 4. **Catching Bugs Early**

```typescript
<Benefit
  title={benefit.titel} // TypeScript error! Did you mean 'title'?
/>
```

## Conclusion

TypeScript transforms JavaScript from a language where errors hide until runtime into one where many mistakes are caught while you type. The initial learning curve pays dividends through:

- Fewer bugs in production
- Faster development with better tooling
- Easier collaboration on large codebases
- More maintainable code

Start small: add types to function parameters, define prop types for components, and use enums for string constants. As you grow comfortable, explore advanced features like generics, utility types, and conditional types
