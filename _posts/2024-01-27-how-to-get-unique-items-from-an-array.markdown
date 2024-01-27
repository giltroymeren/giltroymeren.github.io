---
layout: post
title: "How to get unique items from an array"
date: 2024-01-27
categories: notes javascript
---

## Challenge

I had a very basic problem recently wherein I had to make a form for data with a dynamic unique enum attribute say "type of business", "club affiliation" or something similar. Data from the API is directly provided in the form and this field, a dropdown, had to be always updated with the latest pool of values.

Basically there is an array of repeated values and we need an array of unique values from the first.

## Solution

I have two solutions that arrive to the same result.

Here is a sample data that mirrors what my original data looked like:

~~~json
const data = [
    {
        "industry": "Law",
        "id": 62683781,
    },
    {
        "industry": "Telecommunications",
        "id": 73385749,
    },
    {
        "industry": "Law",
        "id": 22759989,
    },
    {
        "industry": "Health Services",
        "id": 15438299,
    },
    {
        "industry": "Education",
        "id": 62683781,
    },
  ...
]
~~~

and the expected array should contain the following:

~~~javascript
["Law", "Telecommunications", "Health Services", "Education", ...]
~~~

### Array manipulation

I just made a new array from a set generated from the `industry` attributes. The set is a powerful tool here as it creates and stores unique values from the inputs. 

~~~javascript
const result = Array.from(
    new Set(
      data.map(item => item.industry)
    )
  )
~~~

### `for`-loop

I also wanted to make sure to find another simple way that uses tried-and-tested basic JavaScript tools such as loops.

This solution just maintains an array to insert unique values determined by `indexOf`. If the value is not found in `results`, that means it is not yet found and should return `-1`.

There exists a similar method `includes()` that returns a boolean but the difference is that this one returns `true` for the `NaN` value which is unpredictable hence the decision to use the first.

~~~javascript
const results: string[] = []
for(let i = 0; i < data.length; i++) {
  if(results.indexOf(data[i].industry) === -1) {
    results.push(data[i].industry)
  }
}
~~~


## References

- "Array.prototype.includes() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/includes)>.
- "Array.prototype.indexOf() - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/indexOf)>.
- "Set - JavaScript \| MDN." _MDN Web Docs_, Mozilla Developer Network, <[`developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set)>.