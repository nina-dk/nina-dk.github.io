---
title: Why the forEach JavaScript Method Might not Always be the Best Choice
date: 2021-04-20 20:16:04
categories:
- JavaScript
- Array
tags:
- forEach
---

This is a trap I’ve found myself falling into several times.

{% img https://images.unsplash.com/photo-1506702315536-dd8b83e2dcf9?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1170&q=80 %}

Although the `Array.prototype.forEach()` method seems like a good pick when you want to loop over an array of elements in JavaScript, there are several occasions when a simple `for` loop or another one of the built-in array methods is what you need.

<!-- more -->

Let’s create a simple example of a bad use case of the `forEach` method:

{% codeblock lang:javascript %}
let array = [1, 2, 3];
array.forEach(num => num += 1);
{% endcodeblock %}

`forEach` accepts a callback function as an argument (`num => num += 1`), which in turn has one parameter, `num`. This parameter will be assigned to the value of the element that’s being iterated over on each iteration of `forEach`. So, given the array `[1, 2, 3]`, the numbers `1`, `2`, and `3` will be assigned to the parameter `num` of the callback on each iteration.

Now might be a good time to say that in the case of primitive values (such as numbers) JavaScript is “pass-by-value”. This means that when we’re passing the first element of the array, the number `1` in our example, as an argument to the callback function, what really happens is that a copy of the number `1` is assigned to the variable `num`. The element at index 0 of the calling array and the value of the variable `num` inside the callback function are simply equal; they’re not linked in any way.

Inside the callback, we’re reassigning the value of `num` to the result of `num + 1` (that is we increment its value by `1`). So, `num` will now equal `2` (on the first iteration). However, the values of the elements of the calling array are not going to be affected by this reassignment, because of “pass-by-value” behaviour. The calling array will still contain the elements `1`, `2`, and `3`.

This is a trap I’ve found myself falling into several times, and by writing it down, I’m attempting to not only share it but also wrap my head around it for good. An experienced developer would know just by looking at this example above that something is going wrong. This example, the way it is right now, has no useful return value (`forEach` *always* returns `undefined`) and no side effects; in other words, it’s doing nothing at all.

If the goal was to increment every element in the array we could do so by using the `Array.prototype.map()` method, if we don’t mind having a new array instead of mutating the calling array, or a good old `for` loop, to mutate the original array, like so:

{% codeblock lang:javascript %}
for (let i = 0; i < array.length; i++) {
 array[i] += 1;
}
{% endcodeblock %}

Long story short, you cannot change the values of an array’s elements using a `forEach` loop the way you would with `map`, for example. If that’s your goal, the simplest way is to go for a `for` loop. Note, however, that you can mutate an array’s elements if they are objects themselves.

*Note:* There is, however, a less-common workaround to that issue, using the third parameter of `forEach` which is assigned to the calling array, like so:

{% codeblock lang:javascript %}
array.forEach((_, index, arr) => arr[index] += 1);
{% endcodeblock %}

Some other cases when a `forEach` loop is not appropriate are when you intend to exit the looping mechanism after a condition is met or when you want to mutate the calling array while looping through it. In the first case, `forEach` wouldn’t work at all since there’s no way to exit it, and in the latter, you might get unexpected results if the mutation affects the length property of the calling array.

For example:

{% codeblock lang:javascript %}
let nums = [1, 2, 5, 7, 6];
nums.forEach((num, index) => {
  if (num % 2 === 1) {
    nums.splice(index, 1);
  }
});
nums; //returns [ 2, 7, 6 ]
{% endcodeblock %}

In the example above, we’re attempting to remove every odd number from the `nums` array using the `Array.prototype.splice()` destructive method. However, as we can see from the final return value, `nums` still contains the odd number `7`. This happens because while the `forEach` loop iterates over the array, each element and its corresponding index are passed as arguments to the callback function. When an element is removed, the next element will take up the removed element’s slot and therefore, its index. On the next iteration of `forEach` the value assigned to `index` will be incremented as the iteration proceeds with the next element, ignoring the element that was moved back by one slot.

This is better explained in the flowchart below:

{% asset_img forEachBug.png %}