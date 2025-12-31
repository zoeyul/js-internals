## classes 라이브러리 구현

- 여러 타입의 인자(string/object/array 등)을 받아서 조건에 맞는 class 이름만 골라 공백으로 연결된 하나의 문자열을 반환하는 함수 classNames 구현

```js
<div
  className={classNames("btn", { active: isActive }, { disabled: isDisabled })}
/>
```

```js
/**
 * @param {...(any|Object|Array<any|Object|Array>)} args
 * @return {string}
 */
export default function classNames(...args) {
  const classes = [];

  args.forEach((arg) => {
    //ignore falsy values
    if (!arg) {
      return;
    }

    const argType = typeof arg;

    //handle string and numbers
    if (argType === "string" || argType === "number") {
      classes.push(arg);
      return;
    }

    //handle arrays
    if (Array.isArray(arg)) {
      classes.push(classNames(...arg));
      return;
    }

    //handle objects
    if (argType === "object") {
      for (const key in arg) {
        //only process non-inherited keys
        if (Object.hasOwn(arg, key) && arg[key]) {
          classes.push(key);
        }
      }

      return;
    }
  });

  return classes.join("");
}
```

```ts
export type ClassValue =
  | ClassArray
  | ClassDictionary
  | string
  | number
  | null
  | boolean
  | undefined;
export type ClassDictionary = Record<string, any>; //key->class 이름, value->포함 여부 조건
export type ClassArray = Array<ClassValue>; //중첩 가능, 재귀 필요

//모든 args
export default function classNames(...args: Array<ClassValue>): string {
  const classes: Array<string> = [];

  function classNamesImpl(...args: Array<ClassValue>) {
    args.forEach((arg) => {
      // Ignore falsey values.
      if (!arg) {
        return;
      }

      const argType = typeof arg;

      // Handle string and numbers.
      if (argType === "string" || argType === "number") {
        classes.push(String(arg));
        return;
      }

      // Handle arrays. 배열은 새로운 규칙이 아닌 안에 있는 요소를 다시 classnames 규칙으로 처리
      if (Array.isArray(arg)) {
        for (const cls of arg) {
          classNamesImpl(cls);
        }

        return;
      }

      // Handle objects.
      if (argType === "object") {
        const objArg = arg as ClassDictionary;
        for (const key in objArg) {
          // Only process non-inherited keys.
          if (Object.hasOwn(objArg, key) && objArg[key]) {
            classes.push(key);
          }
        }

        return;
      }
    });
  }

  classNamesImpl(...args);

  return classes.join(" ");
}
```
