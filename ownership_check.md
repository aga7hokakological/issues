Consider following code:
```rust
// lib.rs

use anchor_lang::prelude::*;

declare_id!("YourProgramIDGoesHere"); // Replace this with your actual program ID

#[program]
pub mod missing_owner_check {
    use super::*;

    pub const PROGRAM_ID: Pubkey = id!("YourProgramIDGoesHere"); // Replace this with your actual program ID

    #[derive(Accounts)]
    pub struct MissingOwnerCheck<'info> {
        #[account(signer)]
        pub owner: AccountInfo<'info>, // Account to check the owner
    }

    #[error]
    pub enum ErrorCode {
        #[msg("Incorrect owner provided")]
        IncorrectOwner,
    }

    pub fn check_owner(ctx: Context<CheckOwner>, expected_owner: Pubkey) -> ProgramResult {
        // Verify if the owner provided matches the expected owner
        msg!("You can access the message")

        Ok(())
    }
}
```

```rust
pub fn check_owner(ctx: Context<CheckOwner>, expected_owner: Pubkey) -> ProgramResult {
        // Verify if the owner provided matches the expected owner
        if *ctx.accounts.owner.key != expected_owner {
            return Err(ErrorCode::IncorrectOwner.into());
        }
        msg!("You can access the message")

        Ok(())
    }

```
