---
title: "GoLang Slice as Key in Map"
date: 2022-09-25T01:40:25+05:30
tags: ['golang', 'slices', 'maps']
---

<span style="font-size:25px; font-family:'Kalam'">
<br>Whenever I try my hands on competetive programming, my brain's first approach to solve any complex problem is by using slices & maps. I personally find these two Data Structures as the most comforting and efficient.

During my 'new to DS' days, I tried multiple attempts to use a slice as a key in a map, which obviously failed. 
![Compile error](/images/post/slice_as_key_error.png)
In order to find the reason for my failures in using the most easiest option (in my eyes), I googled "Why slice can't be used as a key in map", and it landed me to the following statement:
"A slice can't be a key in map, but an array can be."

This statement increased my curiosity exponentially. If slice is basically referring to an array at the backened, then why array can be a key but slice can't be? To find answers to these questions, I initiated my research (could not find any relevant blog then).

[GoLang's Map spec](https://go.dev/ref/spec#Making_slices_maps_and_channels) strictly mentions the requirement for a key, i.e.<br>

**The comparison operators '==' and '!=' must be fully defined for operands of the key type; thus the key type must not be a function, map, or slice.**

The above statement makes it quite clear, that a slice can't be a key, because two **slices can't be compared** using comparison operators. The next question that pops is "Why slices can't be compared directly when arrays can be?" 
<br>[Go Slices: usage and internals](https://go.dev/blog/slices-intro) helped me understand, how slice is just a descriptor of an array, that mainly consists of: Pointer to an array, Length of the segment, Capacity (maximum length of the segment).

Whenever we declare a slice using make(), make allocates an array and returns a slice pointing to that array.

![Slice_Array](/images/post/slice_array.png)

As slice refers to an array, each slice could/couldn't be pointing to a different array or segments of array. Due to this, a single '==' is not enough to determine if slices are equal or not. (We can obviously compare slices using methods like *'reflect.DeepEqual()'*, but it needs more than just '==').

But the story does not end here, **slice's mutability** is another reason that makes it an unfit cadidate to be a key. Go's Map uses hashmap implementation, so, mutating a value will obviously change the hashmap, which contradicts one of the requirements of a key.

Conclusion of the story :), slices can not be used as key in maps due to the following reasons:
- Slices are mutable, and 
- Could not be compared. 
</span>
