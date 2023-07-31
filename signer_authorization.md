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

```
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

```
 pub fn my_program_accounts(ctx: Context<MyProgramAccounts>) -> ProgramResult {
            // Your logic to handle the instruction here
            msg!("GM");
            Ok(())
        }
```

The best solution to avoid this type of issue we can chane following piece of code
```
#[derive(Accounts)]
pub struct MyProgramAccounts<'info> {
    #[account(init, payer = user, space = 8 + 16)]
    pub my_account: Account<'info, MyAccount>,
    #[account(mut)]
    pub user: Signer<'info>,
}
```
