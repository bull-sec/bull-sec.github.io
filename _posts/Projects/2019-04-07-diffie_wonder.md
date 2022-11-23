---
title: "Blog: The Beauty of Diffie-Hellman"
date: 2022-11-23 11:22 +0100
category: Cryptography
tags: cryptography encryption mathematics
published: true
---

I've never been the greatest mathematician, as a kid I hated the subject with a passion and it's only as I've gotten older I've come to appreciate how useful mathematics can be. With that in mind I've recently become quite interested in Cryptography and the actual maths behind the algorithms, so I sat down this afternoon and decided to try and get a better understanding of how the Diffie-Hellman key exchange works.

For those who are unfamiliar with the Diffie-Hellman key exchange you may be suprised to hear that you probably used it when your browser encountered the SSL certificate on this very website. Diffie-Hellman, named after [Whitfield Diffie](https://en.wikipedia.org/wiki/Whitfield_Diffie) and [Martin Hellman](https://en.wikipedia.org/wiki/Martin_Hellman)  is used to agree a "secret key" between two parties who have not met before, generally you can think of these two parties as your computer and the server you are trying to connect to. Before the two parties can exchange encrypted traffic they must first decide on the key which will be used as the "cipher key". This is where DH comes in:

Using Modular arithmetic we can create a formula which will result in both parties generating the same value, which can be used as their "secret key"

```bash
A: 787 + 89 = 876 
B: 787 + 317 = 1,104 

B: 1104 mod 787 = 317 
A: 876 mod 787 = 89 

A: 317 + 876 = 1,193 
B: 1104 + 89 = 1,193 
```

So what just happened there?

- Both of our parties agreed on the number 787 as their initial value P. (the real number would be a lot larger but 787 will do for demo purposes)

- They both went off and added their own value to P to create Pa & Pb respectively.

- Pa, and Pb are exchanged between the parties and that is where the magic happens.

- A performs modular (or clock) arithmetic on Pb using the initial value P as the terminator and produces the value K.

- B performs the same calculation using Ia and due to the way in which modular arithmetic works, will arrive at the same value of K

```bash
P + A = Ka
P + B = Kb

Pb % P = Kb
Pa % P = Ka

Ka + Pb = K
Kb + Pa = K
```

Now A knows B's secret, and B knows A's secret, and the two share the secret K.

Without knowing what the initial value, plus the additional value added by each party it will be very difficult for an attacker C to work out what the "secret key" K will be.

> It should be noted (again) that the numbers used to calculate the real Diffie-Hellman key exchange are far more complex than the basic primes which were used here as an example.
{: .prompt-info }

The beauty of this algorithm is in it's simplicity. A and B can broadcast their additional values in the clear, and since they both know the original value used it is a simple matter of performing the calculation to work out the shared key. Attacker C having intercepted the additional values in the clear won't know this initial value, and thus will be forced to perform brute force attacks to work out every possible combination to produce the shared key. A feat which becomes increasingly difficult as the number of digits used to create the key grows.

/MB
