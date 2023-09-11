In Solidity, integers are represented using a fixed number of bits. For example, uint8 uses 8 bits and can store values from 0 to 255. 
When an integer is incremented beyond its maximum value or decremented below its minimum value, an overflow or underflow occurs, respectively.
An integer overflow occurs when the result of an arithmetic operation exceeds the maximum value that can be stored in the integer's bit representation. 
For example, if we add 1 to the maximum value of uint8, which is 255, the result is 0, causing an integer overflow.
On the other hand, an integer underflow occurs when the result of an arithmetic operation is less than the minimum value that can be stored in the integer's bit representation. 
For example, if we subtract 1 from the minimum value of int8, which is -128, the result is 127, causing an integer underflow.

Rust programming language is used for writing programs on solana.
Even though rust is considered to be memory safe it cannot prevent overflow/underflow issues by itself and if developer ignores this then it can be quite dangerous for the protocol.
Both Solana smart contracts and Solana’s core runtime (the validator code) written in Rust have seen quite a number of arithmetic errors.

In Rust, an unsigned integer can have one of the following types: u8, u16, u32, u64, u128 and usize. 
The type denotes the number of bits used to store the integer: u8 can hold values between 0 and 255, u16 can hold values between 0 and 65535, and so forth.

> For example, x = x + 1— if an u8 integer x is changed to a value outside of its range, say 256 or -1, then an overflow/underflow will occur.

Type casting is a way of changing one data type to another. Casting a value can change the way we can use the variables in program.
Casting data types can create issues related to overflow and underflow.


How to prevent arithmetic errors in Solana smart contracts (sec3).
When writing a Solana smart contract in Rust, there are three common ways to deal with arithmetic errors:

> Replace+ - * / ** with checked_add checked_sub checked_mul and checked_div checked_pow , respectively. 
> Since the cost of these checks is almost negligible in Solana (compared to high gas fees on Ethereum), it might be a good idea to always have these checks.

> Replace + - * ** with saturating_add saturating_sub saturating_mul and saturating_pow , respectively. This ensures the computed value will stay at the numeric bounds instead of overflowing/underflowing.

> Alternatively, turn on the overflow-checks in release mode by setting overflow-checks = true under [profile.release] . (credit: freax13)

> Cautious on integer division /. To avoid losing precision, change integer to floating point types.
> It is not recommended to deal with money in floating point, because floating point cannot accurately represent most base 10 numbers and errors can accumulate over time. 
> Instead, use a proper decimal type. The article explains nicely the underlying reasons. (credit: Plasma_000 and simukis)
