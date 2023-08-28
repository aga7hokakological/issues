SIGNER AUTHORIZATION:

Solana Anchor provides a simplified and convenient way to handle signer authorization in your Solana programs. Signer authorization ensures that only authorized accounts can execute specific instructions within your program. In Solana, instructions are equivalent to transactions, and signer authorization is a critical security feature to prevent unauthorized access and actions on the blockchain.

Here's how Solana Anchor's signer authorization works:

1. Signer Annotation:
In your Anchor program, you can use the `#[account(signer)]` annotation within the `#[derive(Accounts)]` attribute to specify that a particular account is expected to be a signer for the instruction. This annotation tells Anchor to automatically check if the given account is a valid signer when the instruction is executed.

   For example:
   ```rust
   #[derive(Accounts)]
   pub struct MyInstruction<'info> {
       #[account(signer)] // Indicates that this account must be a signer
       signer_account: AccountInfo<'info>,
       // ... other account fields
   }
   ```

2. Automatic Signer Verification:
When you define an instruction with the `signer` annotation, Anchor takes care of verifying whether the provided account is a valid signer for the transaction. You don't need to manually perform the signature check; Anchor handles it transparently for you.

3. Missing Required Signature:
If the account provided as a signer is not included as a signer in the transaction, Solana will throw a `MissingRequiredSignature` error. This helps prevent unauthorized execution of instructions.

4. Better Security and Readability:
Solana Anchor's signer authorization simplifies the code you need to write and enhances the security of your program. By using the `signer` annotation, you explicitly indicate which accounts require signing authority, making your code more readable and maintaining the security of your program.

5. Protecting Sensitive Operations:
Signer authorization is particularly useful when you want to restrict certain operations or data modifications to authorized users. For example, you might require signer authorization for transferring tokens, updating account data, or executing any other operation that requires verified identity.


Let's define `lib.rs` file

```rust
// lib.rs
use anchor_lang::prelude::*;

declare_id!("YourProgramIDHere");

#[program]
pub mod my_program {
    use super::*;

    // Define the program state
    pub struct MyProgram;

    impl MyProgram {
        // Initialize the program
        pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
            // Your initialization logic here
            Ok(())
        }
    }

    // Define the Initialize instruction
    #[derive(Accounts)]
    pub struct Initialize {}
}
```

Assume the following code in `instruction.rs`

```rust
// instructions.rs
use anchor_lang::prelude::*;

#[derive(Accounts)]
pub struct MyProgramAccounts<'info> {
    #[account(init, payer = user, space = 8 + 16)]
    pub my_account: Account<'info, MyAccount>,
    #[account(mut)]
    pub user: AccountInfo<'info>,
}

#[account]
pub struct MyAccount {
    // Define your account data fields here
}

```

Now let's update `lib.rs`

```rust
// lib.rs
use anchor_lang::prelude::*;

#[program]
pub mod my_program {
    use super::*;

    // Define the program state
    pub struct MyProgram;

    impl MyProgram {
        // Initialize the program
        pub fn initialize(ctx: Context<Initialize>) -> ProgramResult {
            // Your initialization logic here
            Ok(())
        }

        // Handle the MyProgramAccounts instruction
        pub fn my_program_accounts(ctx: Context<MyProgramAccounts>) -> ProgramResult {
            // Your logic to handle the instruction here
            msg!("GM");
            Ok(())
        }
    }

    // Define the Initialize instruction
    #[derive(Accounts)]
    pub struct Initialize {}
}
```
The issue with above code is that below function can be called by anyone without authorization

```rust
 pub fn my_program_accounts(ctx: Context<MyProgramAccounts>) -> ProgramResult {
            // Your logic to handle the instruction here
            msg!("GM");
            Ok(())
        }
```

You can get rid of the vulnerability by adding following lines of code in main program
```rust
if !ctx.accounts.user.is_signer {
            return Err(ProgramError::MissingRequiredSignature);
        }
```

The best solution to avoid this type of issue we can change following piece of code as:
```rust
#[derive(Accounts)]
pub struct MyProgramAccounts<'info> {
    #[account(init, payer = user, space = 8 + 16)]
    pub my_account: Account<'info, MyAccount>,
    #[account(mut)]
    pub user: Signer<'info>,
}
```

In Solana Anchor, both `AccountInfo` and `Signer` are types used to represent accounts, but they serve different purposes:

1. `AccountInfo`:
   - `AccountInfo` represents a general Solana account. It is used to access and manipulate account data on the blockchain.
   - This type is typically used for read and write operations on accounts and contains various metadata, such as the account's address, data, owner, and more.
   - When you want to interact with an account, you'll use `AccountInfo` to access its data and perform actions like updating its state or reading its contents.

2. `Signer`:
   - `Signer` represents a specific account that is authorized to sign transactions.
   - In Solana, transactions need to be signed by the relevant accounts to be valid. A signer account is required to provide a cryptographic signature for the transaction.
   - In Anchor, the `Signer` type abstracts away the need for manually checking if an account is a signer. It automatically handles the signer verification for you.
   - When you define an instruction in Anchor that requires a signer, you'll use `Signer` in the `#[derive(Accounts)]` attribute to specify the account that is expected to be a signer. Anchor takes care of ensuring that the transaction includes the necessary signatures during execution.

In summary, `AccountInfo` is used for general account data access and manipulation, while `Signer` is specifically used to represent an authorized account to sign transactions. Using `Signer` in Anchor simplifies the process of checking signer authorization, as it is handled automatically by the framework.

### Let's understand this with example
Consider the following code where the user is allowed to update the value 
The initialize function allows to initialize the account. As for the update function only the user who has deployed the code should be able to run it 
but without signer check anyone can update that value.

```rust
use anchor_lang::prelude::*;

declare_id!("9b9M3AMGMM2yE9P9WgDfyvXqrBvnfcrjAcVvdETymb6n");

#[program]
pub mod missing_signer {
    use super::*;

    pub fn initialize(ctx: Context<Initialize>) -> Result<()> {
        let my_account = &mut ctx.accounts.my_account;
        my_account.data = 0; 
        Ok(())
    }

    pub fn update(ctx: Context<Update>, data: u32) -> Result<()> {
        let my_account = &mut ctx.accounts.my_account;
        my_account.data = data; 
        Ok(())
    }
}

#[derive(Accounts)]
pub struct Initialize<'info> {
    my_account: AccountInfo<'info, MyAccount>,
    user: AccountInfo<'info>,
    system_program: SystemProgram<'info>,
}

#[derive(Accounts)]
pub struct Update<'info> {
    my_account: AccountInfo<'info, MyAccount>,
}

#[account]
pub struct MyAccount {
    data: u32,
}
```

Let's check the results after running the test
```
missing-signer
  ✔ Is Initialize (80ms)
  ✔ update the value  (45ms)
```

Now let's update the code with secure measures:
```rust
#[derive(Accounts)]
pub struct Initialize<'info> {
    my_account: AccountInfo<'info, MyAccount>,
    user: Signer<'info>,
    system_program: SystemProgram<'info>,
}
```

Let's test again:
```
Error: Signature verification failed
```

You can check the code HERE
