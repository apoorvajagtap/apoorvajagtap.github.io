---
title: "GoLang Slice as Key in Map"
date: 2022-09-25T01:40:25+05:30
---

<!-- <font size="4">  -->
<span style="font-size:25px; font-family:'Kalam'">
<br>Whenever I try my hands on competetive programming, my brain's first approach to solve any complex problem is by using slices & maps. I personally find these two Data Structures as the most comforting and efficient.

So, during my 'new to DS' days, I tried multiple attempts to use a slice as a key in a map, which obviously failed. In order to find the reason for my failures in using the most easiest option (in my eyes), I googled "Why slice can't be used as a key in map", and it landed me to the following statement:
"A slice can't be a key in map, but an array can be."

This statement increased my curiosity exponentially. If slice is basically referring to an array at the backened, then why array can be a key but slice can't be? So, to find answers to these questions, I initiated my research (could not find any relevant blog then).

Reading through the GoLang's internal docs, I realised that the primary reasons are:
- Slices are mutable, and 
- Their inability to be compared. 

**| *Slices vs Arrays* |**<br>
As we are quite aware that arrays have a fixed size, whereas slices are dynamic. Go's Map uses hashmap implementation, so, mutating a value will obviously change the hashmap, which contradicts one of the requirements of a key.

**| *How to compare two different slices* |**<br>
[Go Slices: usage and internals](https://go.dev/blog/slices-intro) helped me understand, how slice is just a descriptor of an array, that mainly consists of: Pointer to an array, Length of the segment, Capacity (maximum length of the segment).

Whenever we declare a slice using make(), make allocates an array and returns a slice pointing to that array.

<img src="/img/slice_array.png" align="center">
<!-- ![Slice-Array](/img/slice_array.png) -->

As slice refers to an array, each slice could/couldn't be pointing to a different array or segments of array. Due to this, a single '==' is not enough to determine if slices are equal or not. There are methods like *reflect.DeepEqual()* to compare, but [GoLang's Map spec](https://go.dev/ref/spec#Making_slices_maps_and_channels) strictly mentions:<br>

**The comparison operators '==' and '!=' must be fully defined for operands of the key type; thus the key type must not be a function, map, or slice.**

If you would like to learn more about slice expansion and how exactly length & capacity affects a slice, stay tuned for the next blog!
<!-- </font> -->
</span>
