# object vs map vs set

### object

- for record, JSON(serialize)

### map

- key-value storage that replaces object: dictionary, index, cache
- no limitation of key type: string, object, function
- best for API design(key-value storage), size check, deletion, look up
- has/get/set/delete for standard API

```js
const map = new Map();

const objKey = { id: 1 };
map.set(objKey, "value");

console.log(map.get(objKey)); // "value"
console.log(map.size); // 1
```

- do indexing and use as cache with API response: convert Array to Map index and reduce look up cost and code duplication
- if you have list as an Array, you need find for every look up, but if you have it as a map index, the intention will be more clear

```js
const post = [
  { id: 101, title: "A" },
  { id: 102, title: "B" },
  { id: 103, title: "C" },
];

const postById = new Map(posts.map((p) => [p.id, p]))

console.log(postById.get(102)) // { id : 102, title : "B" }
console.log(postById.has(000))
```

- same goes for Next.js. Maintain the list from server as an Array for UI rendering, if you have to access id for detailed view or edit screen, keeping map index could be more productive.
- If you normalize request key into string and store in map, you can reduce request duplication.

```js
const cache = new Map();

async function fetchWithCache(url) {
  if (cache.has(url)) return cache.get(url);

  const res = await fetch(url);
  const data = await res.json();
  cache.set(url, data);
  return data;
}
```

- map doesn't work for JSON serialization. If you have to store it in local storage you need to use Array.from(map.entries()) to convert.

### set

- duplicate items will be removed
- collection of visited id, visited flag

```js
const tags = new Set(["js", "react"]);
tags.add("next");
tags.add("react");

console.log(tags.has("react"));
console.log(tags.size);
```

- useful when you use it for checkbox selection status. It's more intuitive than array.
- but when you use it in react, to preserve immutability, make new Set and replace it.

```js
let selected = new Set([1, 3]);

function toggle(id) {
  const next = new Set(selected);
  if (next.has(id)) next.delete(id);
  else next.add(id);
  selected = next;
}

toggle(3);
toggle(2);

console.log([...selected]); // [1, 2]
```

### object vs Map vs Set, What would you choose?

![object_vs_map_vs_set](../images/object_vs_map_vs_set.png)

- check key type
  - object for string/symbol
  - map for string/object/function key type and index/caching
  - set for collection of value, do not allow duplication, membership check
- What's best to use in react/next.js logic
