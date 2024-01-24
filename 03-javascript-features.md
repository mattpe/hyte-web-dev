# "Modern" JavaScript features

Some of the features introduced in ECMAScript 6 (ES6) and later versions of JavaScript are not supported by all (old) browsers. However, it's safe to use them in Node.js and Express.js applications, as long as you are using a recent version of Node.js.

Here is introduced some of the features we will be using in this course.

Also, you might need to recap [the first year JavaScript course](https://github.com/ilkkamtk/JavaScript-englis). Especially the [Arrays, objects and functions section](https://github.com/ilkkamtk/JavaScript-english/blob/main/taulukot-funktiot.md).

## Variable declarations

Use `const` for variables that are not reassigned, and `let` for variables that are reassigned. Never use `var`.

Rule of thumb: use `const` by default, and only use `let` if you know that the variable will be reassigned (`const` won't work). Never use `var`.

### Scoping

- Scope: rules for deciding what names are visible in program execution
- Lexical scoping: visibility is defined by the location of definitions in the program
  code
- Global scope: bindings defined outside any function or code block (always whe you have code between curly braces (`{` and `}`) you have a code block)
- Local scope: bindings defined inside a block, including a function. In Javascript `let` and `const` bindings have a local scope. `var` behaves in different manner. Just don't use it.

```js
const a = 6;
let b = 3;
{
  const a = 9;
  const c = 4;
  {
    let c = 1;
    console.log(a + b + c); // --> 13
    b = 2;
  }
  console.log(a + b); // --> 11
}
console.log(a + b); // --> 8
```

## Template Literals

- Template literals are a convenient way to create strings in JavaScript. They provide a more concise syntax for creating strings that contain variables or expressions.
- Template literals are defined using backticks (\`\`).
- They can contain placeholders (enclosed in `${}`) for inserting variables or expressions.

Example:

```javascript
const name = 'Alice';
const greeting = `Hello ${name}!`;
console.log(greeting); // 'Hello Alice!'
```

## Arrow Functions

Arrow functions were introduced in ECMAScript 6 (ES6) and provide a more concise syntax for creating functions. They offer a cleaner and often more readable way to write functions, especially for short, one-line operations. Here's what you need to know about arrow functions:

1. **Introduction and Syntax:**
   - Arrow functions are defined using the `() => {}` syntax.
   - They consist of parameter list (enclosed in parentheses), followed by the arrow `=>`, and then the function body (enclosed in curly braces).
2. **Differences from Regular Functions:**
   - Unlike regular functions, arrow functions do not have their own [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) context. Instead, they inherit the `this` value from the surrounding code.
   - Arrow functions cannot be used as constructors, meaning they cannot be instantiated using the `new` keyword.
3. **Lexical Scoping and 'this' Context:**
   - The lack of their own `this` context in arrow functions can be advantageous. They use lexical scoping, meaning they inherit the `this` value from the enclosing function or context.
   - This can help avoid common pitfalls when dealing with nested functions or callbacks.
4. **Use Cases and Benefits:**
   - Arrow functions are particularly useful for concise, single-expression functions, such as mapping or filtering arrays.
   - They reduce boilerplate code and make the intention of the function clearer.
   - Example use cases: `array.map()`, `array.filter()`, and other cases where a concise function is required.
5. **Examples:**

   ```javascript
   // Regular function
   function multiply(a, b) {
     return a * b;
   }

   // Equivalent arrow function
   const multiply = (a, b) => a * b;

   // Using arrow function with array.map()
   const numbers = [1, 2, 3, 4, 5];
   const squared = numbers.map((num) => num ** 2);
   ```

6. **Considerations:**
   - While arrow functions provide concise syntax, they might not always be suitable for more complex functions that require a dedicated `this` context or more than one statement.

### Parentheses rules

In arrow functions, parentheses play a crucial role in defining parameters and in certain cases, the function body. Here are the key rules to keep in mind:

1. **Parameter List:**
   - When the arrow function has a single parameter, you can omit the parentheses around the parameter list. For example: `(x) => x * 2` can be simplified to `x => x * 2`.
   - When there are no parameters or more than one parameter, you must enclose the parameter list in parentheses. For example: `(x, y) => x + y`.
2. **Function Body:**
   - If the function body consists of a single expression (an expression that results in a value), you can omit the curly braces `{}` around the function body. The return value of the expression will be automatically returned.
   - If the function body requires multiple statements or if you want to include explicit `return` statements, you must enclose the function body in curly braces `{}`.
3. **Examples:**
   - Single parameter, single expression:

     ```javascript
     const double = (x) => x * 2;
     ```

   - Multiple parameters, single expression:

     ```javascript
     const sum = (x, y) => x + y;
     ```

   - Multiple parameters, multiple statements:

     ```javascript
     const multiplyAndLog = (x, y) => {
       const result = x * y;
       console.log(result);
       return result;
     };
     ```

4. **Returning Objects:**
   - When you want to return an object literal directly from an arrow function, you need to enclose the object in parentheses. This is to avoid confusion with the function body being mistaken for a block.

     ```javascript
     const createPerson = (name, age) => ({name, age});
     ```

5. **Implicit Return:**
   - As mentioned earlier, arrow functions with a single expression have an implicit return. This means that the result of the expression is automatically returned without needing the `return` keyword.

     ```javascript
     const double = (x) => x * 2; // Implicit return of x * 2
     ```

Remember that the choice of parentheses depends on the context and the structure of your arrow function. Following these rules will help you create arrow functions that are both concise and clear in their intent.

## EcmaScript Modules (ESM)

Modules are a way to organize, structure, and encapsulate code in JavaScript applications. They allow you to break down your code into smaller, manageable pieces, making it easier to maintain, collaborate, and reuse. ES6 introduced a standardized module system that provides a clean and efficient way to work with modules.

1. **Exporting Modules:** You can use the [`export` keyword](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) to export functions, variables, classes, or objects from a module.

      ```javascript
      // math.mjs
      export const add = (a, b) => a + b;
      export const subtract = (a, b) => a - b;
      ```

2. **Importing Modules:** The [`import` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) is used to import functionalities from other modules.

      ```javascript
      // index.js
      import {add, subtract} from './math.mjs';
      console.log(add(5, 3)); // 8
      ```

    - **Note** that the .mjs extension is needed

3. **Default Exports:** A module can have a default export, which is the main exported value from the module.

      ```javascript
      // utility.mjs
      export default function capitalize(str) {
        return str.charAt(0).toUpperCase() + str.slice(1);
      }
      ```

      ```javascript
      // index.js
      import capitalize from './utility.mjs';
      ```

4. **Namespace Imports:** You can import all exported values from a module into a single object using the `* as` syntax.

      ```javascript
      // library.mjs
      export const func1 = () => {};
      export const func2 = () => {};
      ```

      ```javascript
      // index.js
      import * as library from './library.mjs';
      library.func1(); // Calling Function 1
      library.func2(); 
      ```

Modules provide a clean and organized way to structure your codebase, making it easier to manage larger projects and promote code reusability. As you become more comfortable with modules, you'll find them invaluable for maintaining a well-organized and maintainable codebase.

### Export Default

The `export default` statement allows you to export a single value as the default export from a module. This value is typically the main or most important functionality that you want to share from the module.

```javascript
// utility.mjs
const greet = (name) => `Hello, ${name}!`;
export default greet;
```

```javascript
// index.js
import greet from './utility.mjs';
console.log(greet('Alice')); // 'Hello, Alice!'
```

In this example, the `greet` function is exported as the default export from the `utility.mjs` module, and it's imported and used in the `index.js` file.

### Named Exports using `export { ... }`

Named exports allow you to export multiple values from a module, each with its own name. This is useful when you want to export multiple functions, variables, or classes from the same module.

```javascript
// math.mjs
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

export {add, subtract}
```

```javascript
// index.js
import {add, subtract} from './math.mjs';

console.log(add(5, 3)); // 8
```

In this example, both the `add` and `subtract` functions are exported as named exports from the `math.mjs` module and imported into the `index.js` file.

#### Combining Default and Named Exports

You can combine `export default` and named exports in the same module to create a flexible and comprehensive module API.

```javascript
// utilities.mjs
export default function getRandomNumber(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

const greet = (name) => `Hello, ${name}!`;
const capitalize = (str) => str.charAt(0).toUpperCase() + str.slice(1);
export {greet, capitalize}
```

```javascript
// index.js
import getRandomNumber, {greet, capitalize} from './utilities.mjs';

console.log(getRandomNumber(1, 100));
console.log(greet('Alice'));
console.log(capitalize('hello'));
```

In this example, the `getRandomNumber` function is exported as the default export, while the `greet` and `capitalize` functions are exported as named exports from the `utilities.mjs` module.

### Browser Compatibility

The ES6 module syntax (import/export) is natively supported by modern web browsers. However, there are some considerations to keep in mind:

1. **Modern Browsers:**
    - Current versions of major browsers, such as Chrome, Firefox, Safari, and Edge, provide native support for ES6 modules.
    - You can freely use the import/export syntax in these browsers without any issues.

2. **Module Type in Script Tag:**
    - When using ES6 modules in a browser environment, you need to specify the `type="module"` attribute in the script
      tag.

      ```html
      <script type="module" src="app.js"></script>
      ```

    - In this case the `defer` attribute is not required, since the processing of the script contents is deferred.

### Node.js Compatibility

Node.js introduced support for ES6 modules starting from version 12. Before that, Node.js only supported the CommonJS module syntax (require/module.exports). Here are some important considerations when using ES6 modules in Node.js:

- To use ES6 modules, you need to use the `.mjs` extension for module files, or enable ESM in `.js` files by adding `"type": "module"` to your `package.json`. This is the preferred way in this course.
- Many of the online resources and tutorials for Node.js still use the CommonJS syntax. However, you can use the ES6 module syntax in Node.js as long as you are using a recent version of Node.js (12 or later).
- You can still use CommonJS `require` and `module.exports` syntax in Node.js. However, mixing ESM and CommonJS
      syntax in the same file can lead to confusion.

## Error handling

Error handling is an important aspect of writing robust and reliable code. JavaScript provides a built-in `Error` class that can be used to create and throw errors. Errors are handled using the [`try...catch` statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/try...catch), which allows you to catch errors and handle them gracefully.

- `try {...}` block: statements to be executed.
- `catch {...}` block: statements to be executed if an exception is thrown in the try-block.
- `finally {...}` block: statements to be executed after the try-block and catch-block execute. The statements in the finally-block execute even if no catch-block handles the exception.
- [Error](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) objects are thrown when runtime errors occur and they hold data of the exception (name, message, source file, linenumber...)
- [`throw`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/throw) can be used to generate user-defined exceptions
- in JS, terms _Error_ and _Exception_ are basically synonymous
  - in some other languages _Errors_ are considered more serious problems that should occur at all

```javascript
const divideNumbers = (dividend, divisor) => {
    if (divisor === 0) {
        throw new Error("Cannot divide by zero");
    }
    return dividend / divisor;
};

try {
    let result = divideNumbers(10, 0);
    console.log("The result is " + result);
} catch (error) {
    console.error("Caught an error: " + error.message);
} finally {
    console.log("Execution of divideNumbers is complete.");
}
```

1. The `divideNumbers` function checks if the divisor is zero. If it is, it _throws_ an _error_.
1. The `try` block attempts to execute the divideNumbers function.
1. If an error is thrown, the catch block catches it and logs the error message.
1. The `finally` block is executed after the try and catch blocks, regardless of whether an exception was caught or not. It's often used for cleanup or other necessary final steps.

## Destructuring

Destructuring is a feature in JavaScript that allows you to extract values from objects and arrays, and assign them to variables using a concise syntax. It simplifies code by providing a more elegant way to work with structured data.

1. **Array Destructuring:** Destructuring arrays allows you to extract values based on their positions.

      ```javascript
      const [first, second, third] = [1, 2, 3];
      console.log(first); // 1
      ```

2. **Object Destructuring:** Destructuring objects enables you to extract values based on their property names.

      ```javascript
      const person = { name: 'Alice', age: 30 };
      const { name, age } = person;
      console.log(name); // 'Alice'
      ```

3. **Default Values:** You can provide default values during destructuring in case the property is undefined or missing.

      ```javascript
      const { role = 'Guest' } = user;
      console.log(role); // 'Guest'
      ```

4. **Renaming Variables:** Destructuring also allows you to rename variables during extraction.

      ```javascript
      const { name: fullName } = person;
      console.log(fullName); // 'Alice'
      ```

5. **Nested Destructuring:** Destructuring can be used to extract values from nested objects and arrays.

      ```javascript
      const company = { name: 'TechCorp', address: { city: 'Silicon Valley' } };
      const { address: { city } } = company;
      console.log(city); // 'Silicon Valley'
      ```

6. **Function Parameters:** Destructuring is useful for extracting function parameters from objects.

      ```javascript
      function displayPerson({ name, age }) {
        console.log(`Name: ${name}, Age: ${age}`);
      }
      ```

7. **Swapping Variables:** Destructuring provides an elegant way to swap variable values without using a temporary variable.

      ```javascript
      let a = 5, b = 10;
      [a, b] = [b, a];
      console.log(a, b); // 10, 5
      ```

8. **Rest Element:** Destructuring can use the rest element (`...`) to capture remaining elements.

      ```javascript
      const [first, ...rest] = [1, 2, 3, 4, 5];
      console.log(first, rest); // 1, [2, 3, 4, 5]
      ```

Destructuring is a versatile feature that significantly enhances code readability and makes working with complex data structures more intuitive.

## Special Operators

Special operators in JavaScript provide functionality for handling specific operations, making your code more concise and efficient. In this section, we'll explore some of the important special operators and their applications:

### Ternary Operator (`? :`)

- The ternary operator provides a compact way to write conditional statements.
- Syntax: `condition ? expressionIfTrue : expressionIfFalse`
- Example:

```javascript
const age = 20;
const isAdult = age >= 18 ? 'Adult' : 'Minor';
console.log(isAdult); // 'Adult'
```

- Equivalent If-Else Comparison:

```javascript
const age = 20;
let isAdult;

if (age >= 18) {
  isAdult = 'Adult';
} else {
  isAdult = 'Minor';
}

console.log(isAdult); // 'Adult'
```

In both cases, the code checks if the `age` is greater than or equal to 18. If the condition is true, the person is considered an adult; otherwise, they are considered a minor. The ternary operator offers a more compact way to achieve the same result as the if-else statement.

### Spread Operator (`...`)

- The spread operator is used to expand elements of an iterable (like an array or string) into individual elements.
- It's often used for creating shallow copies of arrays and combining arrays.
- Use cautiously inside loops.

```javascript
const originalArray = [1, 2, 3];
const newArray = [...originalArray, 4, 5];
console.log(newArray); // [1, 2, 3, 4, 5]
```

### Rest Operator (`...`)

- The rest operator is used in function parameter lists to collect multiple arguments into a single array.
- It allows functions to accept a variable number of arguments.

```javascript
const calculateSum = (...numbers) => {
  console.log('input values: ', numbers);  //-> input values: [1, 2, 3, 4, 5] (array)
  return numbers.reduce((acc, number) => acc + number, 0);
}
// input parameters is list of numbers, not an array
calculateSum(1, 2, 3, 4, 5); // 15
```

### Logical OR Operator (`||`)

- The logical OR operator (`||`) is used to provide a fallback value when the first operand is falsy (evaluates to false). It's commonly employed to handle default values or to conditionally assign values.
- The operator evaluates the expressions from left to right and returns the first truthy value encountered. If no truthy value is found, it returns the last falsy value.

```javascript
const defaultValue = null;
const value = defaultValue || 'Fallback Value';
```

In the example above, if `defaultValue` is falsy (null, undefined, 0, empty string, etc.), the logical OR operator will return `'Fallback Value'`.

### Optional Chaining Operator (`?.`)

The optional chaining operator provides a concise way to access nested properties of an object without causing an error if any of the intermediate properties are undefined or null. It short-circuits the evaluation and returns `undefined` if any part of the chain is not present.

Example:

```javascript
const person = {
  name: 'Alice',
  address: {
    city: 'Wonderland',
  },
};

const cityName = person.address?.city; // 'Wonderland'

const missingCity = person.address?.country?.city; // undefined
```

In this example, if `address` or `country` were not defined, the optional chaining operator ensures that no error
occurs, and the result is `undefined`.

The `?.` operator can be used in various contexts, including:

1. Accessing nested object properties.
2. Calling methods on objects that might be undefined.
3. Accessing array elements within an object.
4. Chaining multiple optional properties.

The optional chaining operator is particularly useful when dealing with data that may not always be complete or consistent, helping you avoid runtime errors and providing a safer way to access properties and navigate complex object structures.
