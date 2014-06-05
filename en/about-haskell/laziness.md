Laziness
--------

Haskell is a lazy language. This means that it will never do the work, if no one wants the result of it.

### Let's start with C++ 

Assume we want a list of some number of the same IP addresses. We will hardly want that in real life, but this example shows the essence of lazy evaluations really well.

The function in C++, which returns the list of addresses, can look like this:

```c++
typedef std::vector<std::string> IPAddresses;

IPAddresses generate_addresses(size_t howMany, const std::string& address) {
    const IPAddresses addresses(howMany, address);
    return addresses;
}
```

Now we need a function, which receives a given number of addresses from this list and prints them on the screen. For example:

```c++
void take_and_print(size_t howMany, const IPAddresses& addresses) {
    for (size_t i = 0; i < howMany; ++i) {
        std::cout << addresses[i] << std::endl;
    }
}

int main() {
	take_and_print(2, generate_addresses(100, "127.0.0.1"));
}
```

The function `take_and_print` receives a list, generated by the function `generate_addresses`, and then prints the first two addresses from this list.

The output will be:

```bash
127.0.0.1
127.0.0.1
```

Everything would be fine, but we only needed two strings out of 100 generated. The remaining 98 strings were generated for nothing. Time was taken, memory was taken - all for nothing.

This is the consequence of the eager evaluations in C++. The function `generate_addresses` is straightforward. It was told to generate 100 addresses and it generated 100 addresses. If you'll tell to generate a million - you'll get a million. Tell it to generate a billion - OK, a little bit of patience, but you'll get a billion.

At the same time the function `take_and_print` is also straightforward and it doesn't care for the efforts of `generate_addresses`. If it was told to display just the first two elements, it'll do just that. And it doesn't care, how many of them are left, ten or half a billion.

The result of eager evaluations is extra work. But functions in Haskell, unlike their hardworking colleagues in C++, hate extra work.

### That's how it is in Haskell 

Open the file `Main.hs` and rewrite it:

```haskell
main = print (take 2 (replicate 100 "127.0.0.1"))
```

The function `replicate` creates a list of 100 "127.0.0.1" addresses, and the function `take` takes the first 2 addresses from this list (hence the number 2 passed to it as the first argument). The function `print` converts it to string and outputs it. Do not mind the syntactical obscurities of this code. They will be explained in full details in the following chapters.

The trick is that the function `replicate` generates a list of 2 addresses, not 100. Why? Because that is the number necessary for the function `take`.

The function `replicate` is lazy. Despite of the fact, that we asked to create a list of 100 strings, it looks around and thinks: "Hmm, who needs my strings? Aha, the function `take` needs them. How much does it need? Alright, just two. Then why do I need to create a hundred, if it needs only two?! Here are your two strings, be happy!"

Yes, hard work is good and laziness is bad, but in this case I like the lazy function more. It, like a good innovator, does not as much, as it was asked, but as much as it is really necessary. This is the essence of lazy evaluations in Haskell.

Of course, if the appetite of the function `take` increases and it will ask for the first fifty elements instead of the first two, then the function `replicate` will create a list of 50 strings. Just as much as neccesary, not a bit more.

But how can we know, that the function `replicate` creates only as much IP addresses as necessary? What if it is not true? Let's check.

Haskell laziness allows us to operate on infinitely large lists. No, not simply very large, but really infinite. Let's rewrite our example:

```haskell
main = print (take 2 (repeat "127.0.0.1"))
```

The function `repeat` creates an infinitely large list of IP addresses, every element of which will be the passed address `127.0.0.1`. If our hardworking function `generate_addresses` from C++ wanted to become just like its lazy colleague, it will need to become something like:

```c++
IPAddresses generate_addresses(const std::string& address) {
	IPAddresses addresses;
	for (;;) {
		addresses.push_back(address);
	}
	return addresses;
}
```

And this function hangs. The cause of it is the eager nature of the function `generate_addresses`. It was told to generate an infinitely large list - and it will do so until the last breath, because the `for` loop has no end.

But if we build our Haskell project and launch it, then there will be no hangs, and two familiar addresses will be printed on the screen.

That's because the function `repeat` is as lazy and rational, as it `replicate` colleague. Sure, we asked it to generate an infinitely large list, but it will generate the list as big as it is necessary. And if in this case only two strings are necessary - fine, here's your list of two strings. And if you need a list of million strings - you'll get a million.

So the essence of lazy evaluations in Haskell is: it doesn't matter how much you are told to do, because in the end you'll do as much as it is in fact necessary.
